# 06 — Governance

**Pergunta que esta pasta responde:** como auditamos e conferimos conformidade?

## O que muda em relação ao Capítulo 1

No Capítulo 1, a trilha de auditoria registrava usuário, pergunta, resposta e fontes citadas — suficiente para um pipeline de um passo só. Aqui, o agente pode dar múltiplos passos (buscar, calcular, decidir chamar outra ferramenta) antes de responder, então a auditoria precisa registrar também **a sequência de passos**, não só o resultado final. O Bedrock Agent expõe isso nativamente via trace, complementado pelo CloudTrail.

## Riscos mapeados

Ver Seção 6 do caso de uso ([`../02-requirements/caso_de_uso.md`](../02-requirements/caso_de_uso.md)) para a lista completa. Resumo dos dois principais:

- **Agente "decidindo" em vez de sinalizar** — mitigado por guardrail configurado (não apenas prompt), testável de forma automatizada.
- **Orquestração não determinística dificultando auditoria** — mitigado exigindo que cada verificação registre quais ferramentas foram chamadas e em que ordem.

⬜ Evidências (prints de trace, logs estruturados) a serem adicionadas durante a execução.
