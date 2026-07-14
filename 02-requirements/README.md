# 02 — Requirements Engineering

**Pergunta que esta pasta responde:** o que o sistema precisa fazer, e como sei que ele fez certo?

## Conteúdo

[`caso_de_uso.md`](caso_de_uso.md) — especificação completa: histórias de usuário (HU-05 a HU-08) com critérios de aceitação testáveis, requisitos não funcionais mapeados a decisões de arquitetura, contrato das ferramentas (action groups), riscos e conformidade.

## Histórias de usuário deste projeto

| HU | Resumo | Status |
|---|---|---|
| HU-05 | Verificação automática de conformidade de prazo | ⬜ Planejada |
| HU-06 | Cálculo de vencimento a partir de data + regra | ⬜ Planejada |
| HU-07 | Guardrail de escopo de decisão (agente sinaliza, não decide) | ⬜ Planejada |
| HU-08 | Exposição como ferramenta via protocolo MCP | ⬜ Planejada |

## Rastreabilidade

Cada HU aqui referencia diretamente uma decisão de arquitetura em [`../03-architecture/`](../03-architecture/) e um critério de validação em [`../05-evaluation/`](../05-evaluation/). Esse vínculo explícito é o núcleo do que este estudo de caso demonstra: requisito rastreável até a evidência de que foi atendido.
