# RAG & LLM

## Revisão de Conceitos

------------------------------------------------------------------------

# 1. Modelos & IA

## LLM --- Large Language Model

Um **LLM** é um modelo de inteligência artificial treinado para
**compreender e gerar linguagem natural**.

Ele recebe uma pergunta (prompt), processa o contexto disponível e gera
uma resposta textual coerente.

Características principais:

-   Capacidade de compreensão semântica de textos
-   Geração de respostas em linguagem natural
-   Uso de grandes volumes de dados durante o treinamento

Exemplos: modelos GPT da OpenAI.

Observação: Modelos mais recentes tendem a possuir **melhor capacidade
de raciocínio e contexto**, porém **custam mais por uso**.

------------------------------------------------------------------------

## Temperatura

A **temperatura** é um parâmetro que controla o **grau de aleatoriedade
na geração de respostas do modelo**.

Valores comuns:

  Temperatura   Comportamento
  ------------- ------------------------------------------
  0.0           Respostas determinísticas e factuais
  0.3 -- 0.5    Equilíbrio entre criatividade e precisão
  0.7+          Respostas mais criativas e variadas

Em **sistemas RAG**, o objetivo é **precisão baseada em documentos**,
portanto normalmente utiliza-se:

    temperature = 0

------------------------------------------------------------------------

## Embeddings (Vetorização)

**Embeddings** são representações numéricas de textos.

O processo transforma frases ou parágrafos em **vetores de números**,
que capturam o **significado semântico** do conteúdo.

Exemplo conceitual:

    "carro elétrico"
    → [0.12, -0.44, 0.91, ...]

Com isso é possível:

-   Comparar significados entre textos
-   Encontrar conteúdos semelhantes
-   Realizar busca semântica

Embeddings são a **base do mecanismo de recuperação em sistemas RAG**.

------------------------------------------------------------------------

# 2. Processamento de Documentos

## Chunks (Divisão em pedaços)

Documentos grandes são divididos em **pequenos blocos de texto**,
chamados **chunks**, antes de serem vetorizados.

Isso é necessário porque:

-   LLMs possuem limite de tokens
-   Vetores menores melhoram a precisão da busca

Parâmetros comuns:

    chunk_size: 1000 caracteres
    chunk_overlap: 200 caracteres

Benefícios:

-   preservação de contexto
-   maior precisão na recuperação
-   melhor performance da busca

------------------------------------------------------------------------

## Text Splitter

O **Text Splitter** é o componente responsável por **dividir documentos
em chunks**.

Uma abordagem comum é o:

### Recursive Text Splitter

Ele divide o texto respeitando a estrutura natural da linguagem:

1.  parágrafos
2.  sentenças
3.  palavras
4.  corte arbitrário

Isso evita que informações importantes sejam quebradas no meio.

Resultado: **chunks semanticamente coerentes**.

------------------------------------------------------------------------

## Metadados

**Metadados** são informações adicionais associadas a cada chunk.

Eles servem para **rastrear a origem do conteúdo** utilizado na resposta
do modelo.

Exemplos:

-   nome do arquivo
-   número da página
-   data de upload
-   identificador único (UUID)
-   tipo de documento

Exemplo de estrutura:

``` json
{
  "text": "conteúdo do chunk...",
  "metadata": {
    "file": "manual.pdf",
    "page": 12,
    "uuid": "123-abc"
  }
}
```

Isso permite:

-   auditoria das respostas
-   rastreabilidade
-   explicabilidade do sistema

------------------------------------------------------------------------

# 3. Banco de Dados Vetorial

## Vector Database (Banco Vetorial)

Um **banco vetorial** é um banco de dados especializado em **armazenar e
pesquisar vetores**.

Diferente de bancos relacionais que fazem busca por igualdade (`=`), ele
realiza **busca por similaridade semântica**.

Fluxo básico:

Documento → Chunk → Embedding → Vector Database

Exemplo utilizado na aula:

-   **Qdrant**

Ele pode rodar:

-   localmente
-   em container Docker
-   na nuvem

------------------------------------------------------------------------

## Collection (Coleção)

Uma **collection** é uma unidade de organização dentro do banco
vetorial.

Equivalente a uma **tabela em bancos relacionais**.

Ao criar uma collection, definimos:

-   dimensão do vetor (ex: 1536)
-   métrica de similaridade
-   estrutura de metadados

Exemplo conceitual:

    collection: documentos_ia

Uma instância do banco pode conter **várias collections para diferentes
projetos**.

------------------------------------------------------------------------

## Similaridade por Cosseno

A **similaridade por cosseno** mede o **grau de proximidade semântica
entre dois vetores**.

Em vez de medir distância, ela calcula o **ângulo entre os vetores**.

Valores possíveis:

  Valor   Significado
  ------- --------------------------
  1       textos muito semelhantes
  \~0.5   relação moderada
  0       pouca ou nenhuma relação

Por isso é uma métrica muito usada para **busca semântica em textos**.

------------------------------------------------------------------------

## Upsert

**Upsert** é uma operação que combina:

Update + Insert

Funcionamento:

-   se o registro **já existir → atualiza**
-   se **não existir → insere**

Exemplo de uso em pipeline de ingestão:

Documento → Chunk → Embedding → Upsert no banco vetorial

Benefícios:

-   evita duplicação de registros
-   simplifica atualizações
-   mantém dados sempre sincronizados

------------------------------------------------------------------------

# Fluxo Completo de um Sistema RAG

Documento\
↓\
Text Splitter\
↓\
Chunks\
↓\
Embeddings\
↓\
Vector Database\
↓\
Busca por Similaridade\
↓\
Contexto Recuperado\
↓\
LLM\
↓\
Resposta Final

Esse processo permite que o LLM **responda perguntas utilizando
conhecimento específico contido nos documentos**.
