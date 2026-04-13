## Resumo dos Conceitos de DDD Estratégico

A aula aborda como analisar um código monolítico desorganizado usando **Domain-Driven Design Estratégico** para identificar como agrupar e separar responsabilidades.

**Domínio** é o negócio principal da aplicação. No exemplo da aula, o domínio é um serviço de streaming. Dentro dele existem **subdomínios**, que são as diferentes áreas do negócio: Billing, Content, Identity/Access, etc.

Os subdomínios se classificam em três tipos: **Core domain** (o diferencial competitivo — no caso, o streaming em si), **Supporting subdomain** (necessário mas não é o diferencial — gerenciamento de usuários, autenticação) e **Generic subdomain** (pode até ser terceirizado/outsourced).

**Bounded Context** é a fronteira linguística onde as palavras têm significados específicos. Um mesmo conceito muda de nome dependendo do contexto: "User" no contexto de Identity é um usuário com credenciais, mas no contexto de Billing é um "Customer" com dados de pagamento. Essa separação é fundamental — cada contexto tem seu próprio vocabulário.

**Ubiquitous Language** (linguagem ubíqua) é o vocabulário comum dentro de cada bounded context. Dentro do contexto de Content, fala-se em "vídeo", "episódio"; dentro de Billing, fala-se em "subscription", "invoice". Quando as pessoas de diferentes áreas da empresa chamam a mesma coisa por nomes diferentes, isso indica bounded contexts separados.

**Coesão** mede o quanto os elementos agrupados juntos são realmente relacionados. Baixa coesão significa ter coisas próximas (na mesma pasta, classe ou módulo) que não são relacionadas entre si — mudam por motivos diferentes e não compartilham o mesmo vocabulário de negócio. A aula propõe medir coesão com critérios como: compartilham o mesmo vocabulário de negócio? Têm relacionamentos diretos? Os arquivos mudam juntos no histórico do Git? São usados juntos nas mesmas operações?

A ideia central é: antes de refatorar, primeiro **identificar** os domínios e subdomínios no código, **medir a coesão** de cada agrupamento, e só depois decidir como reorganizar — seja em módulos, feature folders ou microsserviços.

https://github.com/tech-leads-club/enterprise-apps-classes/tree/c9-s3/docs
