# Projeto 2 — Estudo de Caso: Agente de Verificação de Conformidade Regulatória

**Natureza:** Estudo de caso demonstrativo, construído sobre a base do Projeto 1 (mesma agência fictícia ANVF, mesmo corpus documental). Evolui de "responder perguntas sobre documentos" para "verificar e sinalizar conformidade entre documentos, com ação".

**Papel:** Analista de Requisitos / Engenheiro de Requisitos, especificando um sistema de IA agêntica

**Domínio:** Verificação de conformidade regulatória, com orquestração multi-passo

**Entrega:** Especificação de agente (histórias de usuário, RNFs, guardrails como requisito de negócio), arquitetura com Bedrock Agent, action groups em Python, e exposição via protocolo MCP

---

## 1. Por que este projeto existe

O Projeto 1 provou que um sistema RAG consegue responder perguntas citando a fonte certa, inclusive escolhendo a versão correta entre documentos que se contradizem. Mas ele tem um limite estrutural: só responde o que já está formulado como pergunta direta. Ele não **verifica** nada sozinho, não **calcula** nada, e não teria como, porque `RetrieveAndGenerate` é um pipeline de um passo só (busca, depois gera).

A HU-03 do Projeto 1 (verificação de consistência entre documentos) ficou marcada como "validada parcialmente" exatamente por isso: o sistema acertava porque os documentos se citavam entre si na busca semântica, não porque ele realmente comparou uma regra com um caso concreto.

Este projeto resolve essa lacuna de propósito, com a ferramenta certa para o problema: um agente que decide quais passos dar, não um pipeline fixo.

## 2. Cenário de negócio

Um analista de conformidade da ANVF recebe pareceres técnicos já emitidos e precisa confirmar duas coisas antes de arquivar cada um: (1) o prazo de validade concedido está de acordo com a regra da classe de risco do produto, segundo o manual vigente; e (2) se o produto tem alguma retificação posterior, se a contagem de vigência foi aplicada corretamente.

Hoje isso é feito manualmente, comparando parecer e manual lado a lado. É o tipo de tarefa que consome tempo de analista sênior em verificação repetitiva, não em julgamento — candidata natural a automação assistida, mantendo a decisão final humana.

**Regra de negócio central, herdada do Projeto 1 e reforçada aqui como configuração, não só texto:** o agente verifica e sinaliza; o analista decide. Isso deixa de ser apenas uma frase na especificação e passa a ser um **guardrail configurado**, que bloqueia qualquer tentativa de o agente emitir uma decisão regulatória em nome da agência.

## 3. Requisitos funcionais (histórias de usuário)

**HU-05 — Verificação automática de conformidade de prazo**
Como analista de conformidade, quero que o agente confirme se o prazo de validade de um parecer está de acordo com a regra da classe de risco do produto, para identificar divergências sem comparar documentos manualmente.
- Dado um parecer que cita corretamente o prazo da sua classe de risco, quando verificado, então o agente confirma conformidade e cita as duas fontes comparadas (parecer + manual).
- Dado um parecer cujo prazo diverge da regra padrão sem justificativa registrada, quando verificado, então o agente sinaliza a divergência, sem tentar decidir se ela é aceitável.

**HU-06 — Cálculo de vencimento**
Como analista de conformidade, quero que o agente calcule a data de vencimento de um registro a partir da data de emissão e do prazo vigente, para não depender de cálculo manual sujeito a erro.
- Dada uma data de emissão e um prazo em meses, quando solicitado, então o agente retorna a data de vencimento calculada, junto com o raciocínio (data base + prazo consultado + regra de contagem aplicada).
- Dado um parecer retificado, quando o cálculo é solicitado, então o agente usa a data de emissão do parecer **original** como base de contagem, não a data da revisão (mesma nuance regulatória testada no Projeto 1, agora como comportamento de ferramenta, não só de resposta textual).

**HU-07 — Guardrail de escopo de decisão**
Como gestor de conformidade, quero que o agente se recuse a emitir decisões regulatórias (aprovar, reprovar, ou determinar validade de um registro), para garantir que a decisão final permaneça humana.
- Dado um pedido para o agente "decidir" ou "aprovar" algo, quando processado, então o agente recusa explicitamente e explica que a decisão cabe ao analista responsável.
- Este comportamento é implementado como **guardrail configurado no Bedrock**, não apenas como instrução de prompt — para que a recusa não dependa da boa vontade do modelo em seguir instrução.

**HU-08 — Exposição como ferramenta via protocolo MCP**
Como integrador de sistemas, quero consumir a verificação de conformidade como uma ferramenta padronizada, para conectar este agente a outros sistemas de IA sem depender de uma API proprietária.
- Dado um cliente compatível com MCP, quando ele lista as ferramentas disponíveis, então `verificar_conformidade` e `calcular_vencimento` aparecem descritas com seus parâmetros e comportamento esperado.

## 4. Requisitos não funcionais e decisões de arquitetura

| RNF | Especificação | Decisão AWS |
|---|---|---|
| Orquestração multi-passo | O agente deve decidir dinamicamente se busca, calcula, ou pede mais informação, em vez de seguir uma sequência fixa | Bedrock Agent (substitui o `RetrieveAndGenerate` fixo do Projeto 1) |
| Ferramentas de negócio | Cálculos e comparações determinísticas não devem ser delegados ao "raciocínio" do modelo, e sim a código auditável | Action groups: duas funções Lambda (`calcular_vencimento`, `verificar_conformidade`), descritas ao agente via schema OpenAPI |
| Guardrail de escopo | A recusa de decidir deve ser garantida por configuração, não por instrução de prompt | Bedrock Guardrails, com política de tópico bloqueando linguagem de decisão/aprovação regulatória |
| Auditabilidade de raciocínio | Cada verificação deve registrar não só a resposta final, mas os passos que o agente tomou para chegar nela | CloudTrail + trace do Bedrock Agent (rastreabilidade nativa de agent steps) |
| Interoperabilidade | A ferramenta deve ser consumível por qualquer cliente de IA compatível, não só por uma API própria | Exposição via protocolo MCP, além do endpoint REST |
| Reaproveitamento de infraestrutura | Não recriar o que já existe e já está provado no Projeto 1 | Mesma Knowledge Base, mesmo corpus sintético (ANVF); o que muda é a camada de orquestração acima dela |

## 5. Contrato de ferramentas (amostra)

`calcular_vencimento(data_emissao: str, prazo_meses: int) -> dict`
Retorna `{"data_vencimento": "YYYY-MM-DD", "raciocinio": "string explicando a regra de contagem aplicada"}`.

`verificar_conformidade(documento_parecer: str, classe_risco: str) -> dict`
Retorna `{"conforme": bool, "prazo_parecer": "string", "prazo_regra_manual": "string", "divergencia": "string ou null", "fontes": ["lista de documentos comparados"]}`.

## 6. Riscos e conformidade

- **Agente "decidindo" em vez de sinalizar:** mitigado por guardrail configurado (HU-07), testável de forma automatizada (enviar pedidos de decisão e confirmar recusa consistente).
- **Cálculo de data incorreto por ambiguidade de qual data-base usar:** já observado como risco real no Projeto 1 (contagem a partir do parecer original, não da revisão); tratado aqui como lógica de código determinística, não mais dependente de o modelo "lembrar" a regra corretamente.
- **Orquestração não determinística dificultando auditoria:** mitigado exigindo que o agente registre, a cada verificação, quais ferramentas chamou e em que ordem (trace nativo do Bedrock Agent), não só a resposta final.
- **Alucinação de política de segurança** (achado real do Projeto 1, Q11: o modelo inventou uma justificativa de guardrail que não existia): motivo adicional para implementar guardrails de verdade neste projeto, em vez de depender do modelo "soar responsável" por conta própria.

## 7. Competências demonstradas

Especificação de requisitos para sistemas de IA agêntica · Guardrails como requisito de negócio configurável, não texto de prompt · Orquestração multi-step com Bedrock Agent e action groups · Protocolo MCP para interoperabilidade · Continuidade e reaproveitamento de arquitetura entre projetos · Tradução de um gap identificado no Projeto 1 (HU-03 parcial) em especificação completa no Projeto 2
