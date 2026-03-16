# Banco Vetorial --- Conceitos Fundamentais

## Banco Vetorial

Um banco vetorial é um tipo de banco de dados projetado para armazenar e
buscar vetores numéricos (embeddings) que representam informações como
textos, imagens ou áudios.

Ele permite encontrar conteúdos semanticamente semelhantes, mesmo que as
palavras sejam diferentes.

Exemplo: Buscar documentos que tenham significado parecido com uma
pergunta do usuário.

------------------------------------------------------------------------

## Embedding

Embedding é a representação numérica de um dado (como texto) em forma de
vetor.

Esse vetor captura o significado semântico da informação.

Exemplo:

Texto: "Como aprender programação"

Vetor (exemplo simplificado): \[0.12, -0.44, 0.98, 0.33, ...\]

Textos com significados semelhantes terão vetores próximos.

------------------------------------------------------------------------

## Similaridade Semântica

É a capacidade de medir o quanto dois conteúdos são parecidos em
significado.

Nos bancos vetoriais isso é feito comparando a distância entre vetores.

Quanto menor a distância, mais similares são os conteúdos.

------------------------------------------------------------------------

## Consulta Vetorial (Vector Search)

Processo de buscar no banco vetorial os vetores mais próximos de um
vetor de consulta.

Fluxo conceitual:

1.  O texto da pergunta é convertido em embedding
2.  O banco vetorial compara esse vetor com os vetores armazenados
3.  Retorna os mais semelhantes

------------------------------------------------------------------------

## Recuperação de Informação

Processo de encontrar dados relevantes dentro de uma base de
conhecimento.

No contexto de IA, isso significa recuperar trechos de documentos
relacionados à pergunta.

------------------------------------------------------------------------

## Contexto

Informações recuperadas da base de conhecimento que serão usadas para
ajudar o modelo de IA a gerar uma resposta.

------------------------------------------------------------------------

## Fluxo Conceitual

1.  Documento é convertido em embedding
2.  O embedding é armazenado no banco vetorial
3.  Usuário faz uma pergunta
4.  A pergunta vira um embedding
5.  O banco vetorial retorna os vetores mais similares
6.  Os conteúdos recuperados podem ser usados como contexto
