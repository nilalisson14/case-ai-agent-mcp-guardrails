# 05 — Evaluation

**Pergunta que esta pasta responde:** como medimos se funciona de verdade?

## Metodologia planejada

Seguindo o mesmo princípio do Capítulo 1 (avaliação determinística, sem depender de bibliotecas de LLM-juiz instáveis), a avaliação deste projeto soma dois tipos de teste:

1. **Correção de ferramenta** — dado um parecer e uma classe de risco, o agente chama `verificar_conformidade` e retorna o mesmo veredito que um analista humano chegaria manualmente?
2. **Recusa de guardrail** — dado um pedido explícito para o agente "decidir" ou "aprovar" um registro, ele recusa de forma consistente, em múltiplas tentativas (testando se o guardrail é confiável, não só o modelo "se comportando bem" por acaso)?

⬜ Pendente — dataset e script a serem criados durante a execução, no mesmo formato do Capítulo 1 (`dataset.json` + `avaliar.py`).

## Achado do Capítulo 1 que motivou este desenho de teste

A pergunta adversarial do Capítulo 1 (produto inexistente) revelou o modelo inventando uma justificativa de política de segurança fictícia para justificar uma recusa correta. Isso é exatamente o tipo de comportamento que o teste de "recusa de guardrail" aqui precisa distinguir: recusa por configuração real (confiável) versus recusa por comportamento aparentemente responsável do modelo (não confiável, porque não é garantido).
