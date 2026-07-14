# 04 — AI Engineering

**Pergunta que esta pasta responde:** como cada peça foi implementada?

## Estrutura planejada

```
action-groups/
  calcular_vencimento.py       Lambda determinística: data de emissão + prazo -> data de vencimento
  verificar_conformidade.py    Lambda: compara parecer x regra do manual, aponta divergência
  openapi-schema.json          Schema que descreve as duas ferramentas ao Bedrock Agent
guardrail/
  guardrail-config.json        Política de tópico bloqueando linguagem de decisão regulatória
mcp-server/
  server.py                    Servidor MCP expondo as duas ferramentas para clientes externos
```

⬜ Pendente — código a ser adicionado conforme a execução avança (roadmap em `roadmap/passo_a_passo.md`, na raiz do repositório).

## Por que Lambda para os cálculos, e não o próprio modelo

Ver decisão registrada em [`../03-architecture/README.md`](../03-architecture/README.md). Resumindo: cálculo de data é determinístico, testável e não deveria depender de um LLM "lembrar" a regra de contagem certa.
