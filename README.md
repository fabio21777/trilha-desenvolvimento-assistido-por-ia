# trilha-desenvolvimento-assistido-por-ia


**Dev Usuário vs Builder** — a distinção fundamental: um usa a IA para resolver tarefas, o outro constrói sistemas que a orquestram. O ideal é dominar os dois.

**Tokens & Context** — a "física" das LLMs: tokens são fragmentos de texto (não palavras), a context window é tudo que o modelo vê numa chamada, e o fenômeno *Lost in the Middle* mostra que o meio do contexto é ignorado — por isso você coloca instruções críticas no início e a tarefa no fim.

**Prompt Engineering** — a tríade Role + Constraints + Exemplos. Role define quem o modelo é, constraints controlam o formato/escopo da saída, e exemplos (few-shot) ensinam o padrão melhor do que qualquer descrição verbal.

**Context Engineering** — vai além do prompt único: você arquiteta camadas (system prompt fixo → conhecimento injetado dinâmicamente → estado resumido → tarefa atual). É exatamente o que você faz com seus arquivos `SKILL.md` e documentos de contexto.

**MCPs** — o protocolo que transforma LLMs de "geradores de texto" em agentes que agem: filesystem, git, banco, AWS, browser. O Claude Code inteiro é construído em cima disso.
