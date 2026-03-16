# trilha-desenvolvimento-assistido-por-ia


**Dev Usuário vs Builder** — a distinção fundamental: um usa a IA para resolver tarefas, o outro constrói sistemas que a orquestram. O ideal é dominar os dois.

Guia atualizado com tudo que faltava. O que foi adicionado/expandido em cada aba:

**Tokens & LLM** — composição visual da context window por camadas coloridas, detalhe sobre truncamento automático ao estourar o limite, e a seção de limitações (sem memória entre sessões, knowledge cutoff, não-determinismo, não executa código nativamente).

**Alucinações** — aba dedicada com os 4 tipos mais comuns no dia a dia de dev (factual, bibliográfico, de código e de dependência/API), explicação de por que acontece estruturalmente, e 5 estratégias práticas de mitigação com a regra de ouro: nunca confie em versões de lib ou nomes de método sem verificar.

**Prompt Engineering** — expandido com Persona (diferente de Role — é o personagem persistente do system prompt), Validação embutida no prompt (equivalente ao Chain-of-Thought) e Few-Shot com exemplo de padrão REST.

**Context Engineering** — as 5 camadas nomeadas e coloridas com exemplo real do Checklist Pro, mais técnicas específicas: RAG, resumo progressivo, chunk ordering e context pruning.

**MCPs & Agentes** — conceito de Agente com o loop ReAct (Reason → Act → Observe → Repeat), exemplo de execução passo a passo, e os 3 tipos de agente (single, multi-agent orquestrador, RAG agent) com alerta sobre falhas e necessidade de fallback.
