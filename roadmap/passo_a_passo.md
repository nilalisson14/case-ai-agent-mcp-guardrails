# Passo a passo — Projeto 2 (Agente de Conformidade + MCP + Guardrails)

Objetivo: transformar o caso de uso especificado em `02-requirements/caso_de_uso.md` em implementação de referência, reaproveitando o corpus e os aprendizados do Capítulo 1. Orçamento alvo: mesmo padrão do Capítulo 1, US$ 5-10.

**Status atual:** planejamento concluído, execução não iniciada.

## Fase 0 — Infraestrutura base (reaproveitada, ~20 min)

Conta AWS, MFA, usuário IAM e Budgets já existem da conta usada no Capítulo 1 — não recriar. Confirmar apenas:
1. Budget ainda ativo (Billing → Budgets)
2. Região de trabalho: us-east-1 (mesma do Capítulo 1)

## Fase 1 — Dados (recriar, ~30 min)

A infraestrutura de dados do Capítulo 1 foi desligada ao final daquele projeto. Recriar:
1. Bucket S3 novo (nome sugerido: `nil-case2-agente-docs`)
2. Reupload dos 6 PDFs sintéticos do corpus ANVF (já existem localmente, do Capítulo 1)
3. Knowledge Base nova no Bedrock, mesmo padrão do Capítulo 1 (Titan Embeddings V2 + S3 Vectors)
4. Sincronizar e validar com uma pergunta de teste simples

## Fase 2 — Bedrock Agent (novo, ~1 h)

1. Bedrock → Agents → Create Agent
2. Nome: `nil-case2-agente-conformidade`
3. Foundation model: Nova 2 Lite (mesmo do Capítulo 1)
4. Associar a Knowledge Base criada na Fase 1 como fonte de conhecimento do agente
5. Instrução do agente (system prompt): definir claramente o papel (verificar conformidade, nunca decidir), mesmo sabendo que o guardrail (Fase 4) é quem garante isso de verdade, não o prompt
6. Testar no console: pergunta simples que só precisa da Knowledge Base, sem ferramentas ainda

## Fase 3 — Action groups (novo, ~1-2 h)

1. Lambda 1: `calcular_vencimento` — recebe data de emissão e prazo em meses, retorna data de vencimento e o raciocínio da regra de contagem aplicada
2. Lambda 2: `verificar_conformidade` — recebe referência a um parecer e uma classe de risco, compara com a regra do manual, retorna conformidade ou divergência com as fontes comparadas
3. Schema OpenAPI descrevendo as duas ferramentas
4. Anexar as duas como Action Group do agente
5. Testar cada ferramenta isoladamente antes de testar via agente

## Fase 4 — Guardrail (novo, ~45 min)

1. Bedrock → Guardrails → Create guardrail
2. Política de tópico negado: bloquear linguagem de decisão/aprovação regulatória (ex.: "aprovo o registro", "decido que...")
3. Anexar o guardrail ao agente
4. Testar explicitamente: pedir ao agente para "decidir" ou "aprovar" algo, confirmar recusa consistente em múltiplas tentativas (não uma vez só)

## Fase 5 — Teste de orquestração multi-passo (~1 h)

No console de teste do Agent, validar as 4 HUs:
- HU-05: pergunta que exige checar parecer contra o manual (agente deve chamar `verificar_conformidade`)
- HU-06: pergunta que exige calcular uma data (agente deve chamar `calcular_vencimento`)
- HU-06 (nuance): parecer retificado — confirmar que o cálculo usa a data do parecer original, não da revisão
- HU-07: pedido de decisão — confirmar recusa via guardrail

Evidência: prints do trace do agente mostrando a sequência de ferramentas chamadas em cada caso.

## Fase 6 — Exposição via MCP (novo, ~2-3 h)

1. Servidor MCP em Python (SDK oficial `mcp`), expondo `calcular_vencimento` e `verificar_conformidade` como tools
2. Testar localmente com um cliente MCP (inspector oficial do protocolo, ou Claude Desktop com conector customizado)
3. Documentar a diferença entre o endpoint REST (Fase 2-3, para qualquer consumidor HTTP) e a interface MCP (para agentes/LLMs externos)

## Fase 7 — Auditoria (~45 min)

1. Confirmar que o trace do Bedrock Agent está sendo capturado (nativo)
2. CloudWatch Logs estruturado na Lambda de entrada, registrando usuário, pergunta, ferramentas chamadas em sequência, resultado final
3. Retenção configurada para 30 dias (mesmo padrão do Capítulo 1)

## Fase 8 — Avaliação de qualidade (~2 h)

1. Dataset de perguntas cobrindo os 4 cenários da Fase 5, com gabarito conhecido
2. Script de avaliação determinística (reaproveitar a lógica de `avaliar.py` do Capítulo 1, adaptado para verificar qual ferramenta foi chamada, não só o texto da resposta)
3. Rodar, documentar achados reais (não forçar 100% de primeira — documentar o que aparecer)

## Fase 9 — Empacotar (~1 h)

1. Preencher os READMEs de cada pasta com os resultados reais (hoje têm ⬜ pendente em vários pontos)
2. Prints de evidência de cada fase
3. Posts de acompanhamento no LinkedIn, mesmo padrão do Capítulo 1

## Fase 10 — Desligar infraestrutura (~30 min)

Mesma ordem do Capítulo 1: Agent → Knowledge Base → Lambdas (action groups + entrada) → bucket S3 → roles IAM. Confirmar custo residual no Cost Explorer após 24h.
