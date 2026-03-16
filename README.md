# trilha-desenvolvimento-assistido-por-ia


Segue uma **versão bem simples e direta**, focada apenas em **definições rápidas para revisão**.

---

# IA no Desenvolvimento — Revisão Rápida

## Cenário atual de IA

IA está sendo usada para aumentar a produtividade no desenvolvimento.

Principais usos:

* gerar código
* corrigir bugs
* escrever documentação
* criar testes

Ferramentas comuns:

* GitHub Copilot
* Cursor IDE
* Ollama

---

# Dev Usuário vs Dev Builder

**Dev Usuário**

* usa IA para ajudar no desenvolvimento
* foco em produtividade

**Dev Builder**

* cria aplicações que utilizam IA
* integra LLMs, APIs e agentes

---

# Tokens

Tokens são **unidades de texto usadas pelos modelos**.

Podem ser:

* palavras
* partes de palavras
* símbolos

APIs normalmente cobram por:

* tokens de entrada
* tokens de saída

---

# Context Window

É o **máximo de tokens que o modelo consegue processar em uma conversa**.

Inclui:

* prompt
* histórico
* resposta

Se passar do limite → parte do contexto é perdida.

---

# Attention

Mecanismo usado pelos modelos **Transformer** para entender relações entre palavras.

Ele define **quais partes do texto são mais importantes** para gerar a resposta.

---

# Limitações das LLMs

Principais problemas:

* **Hallucination** → inventar informações
* **Dependência de contexto** → respostas dependem do prompt
* **Sem memória real** → não lembram conversas fora do contexto

---

# Prompt Engineering

Técnica de **escrever prompts de forma estruturada** para melhorar respostas.

Elementos comuns:

**Role**

Define o papel do modelo.

Exemplo:

```
You are a senior backend developer
```

**Constraints**

Define regras ou limitações.

Exemplo:

```
Use Java 21
```

**Output format**

Define formato da resposta.

Exemplo:

```
Return JSON
```

---

# Context Engineering

Organização das informações enviadas ao modelo.

Objetivo:

* melhorar precisão
* reduzir erros

Técnicas comuns:

* **Chunking** → dividir textos grandes
* **RAG** → buscar documentos relevantes antes da resposta

---

# MCP

**Model Context Protocol**

Permite que a IA se conecte a ferramentas externas.

Exemplos:

* APIs
* banco de dados
* arquivos

---

# Agentes de IA

Sistemas que usam LLMs para **planejar e executar tarefas automaticamente**.

Fluxo básico:

```
Pensar → agir → observar resultado
```

Frameworks conhecidos:

* LangChain
* CrewAI

---

# Spec Driven Development

Método onde **uma especificação clara guia a geração de código pela IA**.

Fluxo:

```
Especificação → IA → Código
```

---

# Futuro

Tendências principais:

* IDEs com IA integrada
* automação de tarefas de desenvolvimento
* sistemas com múltiplos agentes de IA
* desenvolvedor atuando mais como **arquiteto e orquestrador**

---

Se quiser, posso também te passar uma **“folha de revisão de 30 segundos” (super curta mesmo)** que muita gente usa **antes de entrevista técnica** para lembrar rapidamente desses conceitos.
