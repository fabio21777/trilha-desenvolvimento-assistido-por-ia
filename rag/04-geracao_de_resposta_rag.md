# Geração de Resposta — RAG com LangChain

---

## System Prompt

Instruções passadas ao LLM antes da conversa para definir seu comportamento.

Define:

- papel do modelo ("você é um assistente que responde apenas com base em documentos")
- idioma da resposta
- regras obrigatórias (ex: se não encontrar, dizer que não sabe)
- formato de citação das fontes

Exemplo de regras comuns:

```
- Use apenas as informações dos documentos para responder
- Se a informação não estiver nos documentos, responda que não encontrou
- Responda em português brasileiro
- Cite as fontes no formato [1], [2]
```

---

## Prompt Template

Estrutura reutilizável com variáveis dinâmicas que organiza as mensagens enviadas ao LLM.

Divide as mensagens em dois blocos:

- **system** → instruções do sistema (comportamento do modelo)
- **human** → contexto recuperado + pergunta do usuário

Criado com `ChatPromptTemplate.fromMessages([...])` no LangChain.

Variáveis substituídas na hora da chamada:

- `{context}` → chunks recuperados do banco vetorial
- `{question}` → pergunta do usuário

---

## Chain (.pipe)

Encadeia componentes em sequência, passando a saída de um como entrada do próximo.

Estrutura típica de uma chain RAG:

```
prompt template → LLM → output parser
```

Construída com o método `.pipe()`:

```ts
const chain = prompt.pipe(llm).pipe(new StringOutputParser())
```

Uma única chamada percorre todo o fluxo e retorna a resposta já formatada.

---

## StringOutputParser

Componente que transforma a resposta bruta do LLM em texto puro.

Descarta metadados internos do modelo e extrai apenas o conteúdo textual final.

É sempre o último elemento da chain.

---

## Context (Contexto de busca)

String formada pela concatenação dos chunks recuperados do banco vetorial.

Construída mapeando os resultados da busca e unindo com quebra de linha:

```ts
const context = searchResults
  .map((item, i) => `[${i + 1}] ${item.text}`)
  .join('\n')
```

É injetada no prompt template via variável `{context}` para que o LLM responda com base apenas nesse conteúdo.

---

## Sources (Fontes)

Lista de referências que acompanha a resposta gerada.

Cada fonte contém:

- `fileName` → nome do arquivo de origem
- `page` → número da página do PDF
- `score` → grau de similaridade do chunk com a pergunta

Permite:

- rastrear a origem de cada informação
- auditar as respostas
- exibir ao usuário de onde veio o conteúdo

---

## Score de similaridade (na recuperação)

Valor numérico retornado pelo banco vetorial junto a cada chunk encontrado.

Indica o quão relevante aquele trecho é para a pergunta feita. Chunks com score baixo podem ter sido recuperados mas não serem os mais adequados.

Útil para avaliar a qualidade da busca e ajustar o Top K.

---

## Top K

Parâmetro que define quantos chunks serão recuperados do banco vetorial por consulta.

| K maior | mais contexto → melhor precisão → mais tokens → mais custo |
|---------|-------------------------------------------------------------|
| K menor | menos contexto → mais econômico → pode perder informações  |

O valor ideal depende da complexidade esperada das perguntas e do tamanho dos documentos.

---

## Token usage

Quantidade de tokens consumidos em cada chamada ao LLM.

Inclui tokens de entrada (prompt + contexto) e tokens de saída (resposta gerada).

Como a cobrança é por token, retornar esse valor na resposta da API permite acompanhar e controlar custos de operação.

---

## LangSmith

Plataforma do LangChain para observabilidade de aplicações com LLMs.

Oferece:

- rastreamento de cada etapa da chain
- prompts enviados e respostas recebidas
- latência por chamada
- custo detalhado por uso

Alternativa mais robusta ao simples log de tokens na aplicação.

---

## Fluxo completo de geração de resposta

```
Pergunta do usuário
  ↓
Vetorizar a pergunta (embedding)
  ↓
Busca semântica no banco vetorial (Top K chunks)
  ↓
Montar contexto (concatenar chunks)
  ↓
Injetar contexto + pergunta no Prompt Template
  ↓
Executar a Chain (LLM + OutputParser)
  ↓
Extrair resposta e formatar fontes
  ↓
Retornar { question, answer, sources, tokensUsed }
```
