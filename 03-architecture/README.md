# 03 — Solution Architecture

**Pergunta que esta pasta responde:** por que esta arquitetura, e não outra?

## Diagrama

Ver [`arquitetura_projeto2.mermaid`](arquitetura_projeto2.mermaid) (renderizado no README principal do repositório).

## Decisões de arquitetura (ADR resumido)

| Decisão | Alternativa considerada | Por que esta e não a outra |
|---|---|---|
| Bedrock Agent, não `RetrieveAndGenerate` fixo | Manter o pipeline simples do Capítulo 1 | O cenário exige decidir dinamicamente entre buscar, calcular ou recusar — pipeline de um passo não permite isso |
| Cálculos em Lambda determinística, não delegados ao modelo | Pedir ao LLM para calcular a data de vencimento | Cálculo de data tem resposta certa e verificável; delegar isso a um LLM reintroduziria o risco de erro já observado no Capítulo 1 (contagem de prazo a partir da data errada) |
| Guardrail configurado no Bedrock, não só instrução de prompt | Confiar em prompt engineering para bloquear "decisões" do agente | O Capítulo 1 (Q11) mostrou o modelo fabricando uma justificativa de política de segurança inexistente — prova de que instrução de prompt sozinha não é confiável para esse tipo de restrição |
| Reaproveitar corpus e Knowledge Base do Capítulo 1 | Criar corpus novo para este projeto | Corpus já validado, com gabarito conhecido; esforço fica concentrado no que é genuinamente novo (Agent, action groups, guardrails, MCP) |
| MCP como requisito desde o início (HU-08) | Deixar para uma fase futura, como no Capítulo 1 | Aqui o caso de uso (expor uma ferramenta de verificação) é o cenário mais natural do protocolo, diferente do Capítulo 1 onde seria apenas uma reexposição de API |

## Contrato das ferramentas

Ver Seção 5 do caso de uso ([`../02-requirements/caso_de_uso.md`](../02-requirements/caso_de_uso.md)).
