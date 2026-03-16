# Revisão — RAG com Node.js e LangChain

---

## RAG (Retrieval-Augmented Generation)

Técnica de fornecer **contexto privado a um LLM** para respostas mais precisas. LLMs só conhecem dados públicos até sua data de treino — RAG preenche essa lacuna com conhecimento interno sem re-treinar o modelo.

**Fluxo em duas etapas:**

**Ingestão** → documento é fragmentado (*chunks*) → cada fragmento é vetorizado → vetores são armazenados no banco vetorial.

**Consulta** → pergunta é vetorizada → busca semântica no banco vetorial → fragmentos relevantes + pergunta são enviados ao LLM → resposta em linguagem natural.

---

## LangChain

Framework que **abstrai a integração com LLMs**. Cada provedor (OpenAI, Anthropic, Gemini…) tem sua própria API — LangChain unifica isso em uma interface única, permitindo trocar de provedor apenas na configuração.

É para IA o que o **Spring é para Java**: padronização, abstração de integrações e reuso de lógica.

---

## Vetorização (Embeddings)

Conversão de texto em **arrays numéricos que representam significado semântico**. Textos com sentido parecido geram vetores matematicamente próximos, viabilizando busca por similaridade — não por palavras-chave.

O **banco vetorial** (ex: Qdrant, ChromaDB, PGVector) é especializado em armazenar e pesquisar esses vetores de forma eficiente.

---
