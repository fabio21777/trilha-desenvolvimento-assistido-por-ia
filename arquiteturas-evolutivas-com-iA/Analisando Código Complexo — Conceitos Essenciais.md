# Analisando Código Complexo — Conceitos Essenciais

## Domínio e Subdomínios

- **Domínio** = o negócio como um todo
- **Subdomínio Core** = o que te diferencia da concorrência (ex: o streaming em si)
- **Subdomínio de Suporte** = áreas necessárias mas genéricas (ex: autenticação, usuários, pagamento)

---

## Coesão

> Coisas que mudam pelas mesmas razões devem estar juntas.

| Nível | Significado |
|---|---|
| **Alta coesão** | Tudo no grupo pertence ao mesmo contexto |
| **Baixa coesão** | Itens agrupados por acidente, sem relação real |

**Score de 0 a 3:**
- `3` — compartilham vocabulário de negócio, mudam juntos, sem dependências externas
- `2` — têm relacionamentos diretos
- `1` — mudam juntos às vezes
- `0` — não têm relação nenhuma

---

## Bounded Context (Fronteira Linguística)

A mesma entidade pode ter nomes diferentes em contextos distintos.

| Contexto | Nome da entidade |
|---|---|
| Identidade / Auth | `User` |
| Cobrança / Billing | `Customer` |
| Conteúdo | `Viewer` |

> Quando o nome muda, o contexto mudou. Essa é a fronteira.

---

## Como Identificar Domínios no Código

1. Listar todas as entidades, serviços e controllers
2. Agrupar pelo **vocabulário de negócio** compartilhado
3. Identificar **onde a linguagem muda** → nova fronteira
4. Verificar quais arquivos **mudam juntos** (histórico Git)
5. Medir o score de coesão de cada grupo
6. Mapear **dependências entre domínios** (acoplamento cruzado é sinal de perigo)

---

## Sinais de Baixa Coesão

- Pasta `service/` com coisas totalmente diferentes
- Uma classe que muda por motivos distintos
- Entidades compartilhadas entre domínios via banco de dados
- Um serviço que depende de muitos outros domínios

---

## Próximos Passos Após a Análise

Com o mapa de domínios e scores em mãos, as opções são:

- **Modularização** — separar em módulos dentro do mesmo projeto
- **Feature Folders** — organizar por funcionalidade de negócio
- **Vertical Slice Architecture** — cada fatia tem tudo que precisa, do início ao fim
