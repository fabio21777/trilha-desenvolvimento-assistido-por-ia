# Padrão Transactional Outbox

## O Problema: Dual Write

Em arquiteturas baseadas em eventos, existe um desafio clássico: como garantir que uma operação no banco de dados **e** a publicação de um evento aconteçam de forma atômica?

Se o commit no banco ocorre mas o broker está fora do ar, o evento se perde. Se o evento é publicado antes do commit e a transação falha, consumidores processam um evento fantasma. Esse é o problema do **dual write** — duas escritas em sistemas distintos sem transação que as una.

```
  Serviço                  Banco               Broker (SNS/SQS)
 ┌────────┐            ┌──────────┐           ┌──────────────────┐
 │ salvar  │───────────►│ INSERT   │           │                  │
 │ pedido  │            │ pedido   │           │                  │
 │         │            │ ✅ OK     │           │                  │
 │         │            └──────────┘           │                  │
 │         │                                   │                  │
 │ publicar│──────────────────────────────────►│ PUBLISH evento   │
 │ evento  │                                   │ ❌ BROKER DOWN!   │
 └────────┘                                   └──────────────────┘

 Resultado: Pedido salvo, evento PERDIDO.
            Consumidores nunca saberão do pedido.
```

---

## A Solução

Não publique o evento diretamente. Grave-o como registro numa tabela `outbox_events` dentro da **mesma transação** que persiste o dado de negócio. Um processo separado (poller) lê essa tabela e publica os eventos no broker.

A atomicidade é garantida pelo próprio banco — se a transação faz rollback, o evento na outbox também desaparece.

```
  Serviço                       Banco de Dados
 ┌────────────┐          ┌──────────────────────────┐
 │            │          │  ┌────────────────────┐   │
 │ aprovar    │─────────►│  │ UPDATE manual      │   │
 │ manual     │          │  │ INSERT outbox_event │   │
 │            │          │  │ COMMIT ✅            │   │
 │            │          │  └────────────────────┘   │
 └────────────┘          │   Transação ÚNICA         │
                         └──────────────────────────┘
                                     │
                                     ▼
                          ┌────────────────────┐
                          │  outbox_events      │
                          │  published = false   │
                          │  aguardando poller   │
                          └────────────────────┘
```

---

## Tabela de Eventos (`outbox_events`)

```sql
CREATE TABLE outbox_events (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type  VARCHAR(100)  NOT NULL,
    aggregate_id    VARCHAR(100)  NOT NULL,
    event_type      VARCHAR(150)  NOT NULL,
    payload         JSONB         NOT NULL,
    published       BOOLEAN       NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMP     NOT NULL DEFAULT now(),
    published_at    TIMESTAMP,
    retry_count     INT           NOT NULL DEFAULT 0,
    last_error      TEXT
);

CREATE INDEX idx_outbox_unpublished
    ON outbox_events (created_at ASC)
    WHERE published = FALSE;
```

```
 ┌──────────────┬──────────────────────────────────────────────────┐
 │ Campo        │ Propósito                                        │
 ├──────────────┼──────────────────────────────────────────────────┤
 │ id           │ identifica unicamente o evento                   │
 │ aggregate_*  │ entidade dona do evento (Manual, Checklist...)   │
 │ event_type   │ tipo do evento (ManualApproved, PedidoCriado)    │
 │ payload      │ corpo JSONB — permite queries sem desserializar  │
 │ published    │ false → true (flag consumido pelo poller)        │
 │ retry_count  │ tentativas de publicação com backoff             │
 │ last_error   │ mensagem do último erro para diagnóstico         │
 └──────────────┴──────────────────────────────────────────────────┘
```

---

## Processo de Polling

O poller é um `@Scheduled` que periodicamente varre a tabela outbox em busca de eventos não publicados.

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                     CICLO DO POLLER                             │
 │                                                                 │
 │  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
 │  │ 1. SELECT    │    │ 2. PUBLISH   │    │ 3. UPDATE        │  │
 │  │ eventos      │───►│ no broker    │───►│ published = true │  │
 │  │ pendentes    │    │ (SNS/SQS)    │    │ published_at     │  │
 │  └──────────────┘    └──────┬───────┘    └──────────────────┘  │
 │                             │                                   │
 │                        ❌ falha?                                 │
 │                             │                                   │
 │                    ┌────────▼────────┐                          │
 │                    │ retry_count++   │                          │
 │                    │ last_error = msg│                          │
 │                    └─────────────────┘                          │
 │                                                                 │
 │  ⏱️  Aguarda 5 segundos → repete                                │
 └─────────────────────────────────────────────────────────────────┘
```

### Intervalo de 5 segundos — trade-off

```
 Intervalo    Latência média    Carga no banco
 ─────────    ──────────────    ──────────────
    1s           ~500ms            Alta
    5s           ~2.5s             Moderada      ◄── recomendado
   30s           ~15s              Baixa
```

Para near-real-time (sub-segundo), considere CDC (Change Data Capture) com Debezium em vez de polling.

---

## Transações Curtas vs. Processamento Assíncrono

Este é o ponto arquitetural central do padrão.

```
 Transação curta           Poller              Consumidores
 ┌─────────────────┐     ┌──────────┐     ┌──────────────────┐
 │ UPDATE manual    │     │ SELECT   │     │ Gerar embeddings │
 │ INSERT outbox    │ ──► │ PUBLISH  │ ──► │ Criar áudio TTS  │
 │ COMMIT           │     │ UPDATE   │     │ Indexar no vector │
 └─────────────────┘     └──────────┘     └──────────────────┘
     ~10ms                  ~5s              ~30s-5min
```

### O que fica DENTRO da transação

```
 ┌─────────────────────────────────────┐
 │  ✅ Validar regras de negócio        │
 │  ✅ UPDATE/INSERT entidade           │
 │  ✅ INSERT evento na outbox          │
 │  ✅ COMMIT                           │
 └─────────────────────────────────────┘
```

### O que fica FORA da transação

```
 ┌─────────────────────────────────────┐
 │  ❌ Chamada HTTP para outro serviço  │
 │  ❌ Publicação no SNS/SQS/Kafka     │
 │  ❌ Geração de PDF ou áudio          │
 │  ❌ Envio de e-mail                  │
 │  ❌ Geração de embeddings            │
 └─────────────────────────────────────┘
```

### Benefícios da separação

```
 ┌──────────────────┬─────────────────────────────────────────────┐
 │ Resiliência      │ Se processamento pesado falha, dado já está │
 │                  │ salvo. Evento pode ser reprocessado.         │
 ├──────────────────┼─────────────────────────────────────────────┤
 │ Performance      │ Transação do usuário responde em ms,        │
 │                  │ sem esperar I/O externo.                     │
 ├──────────────────┼─────────────────────────────────────────────┤
 │ Independência    │ Cada consumidor processa no seu ritmo.       │
 │                  │ TTS não bloqueia embeddings.                 │
 ├──────────────────┼─────────────────────────────────────────────┤
 │ Observabilidade  │ Tabela outbox = log auditável de todos      │
 │                  │ os eventos emitidos pelo sistema.            │
 └──────────────────┴─────────────────────────────────────────────┘
```

---

## Garantias e Limitações

### At-least-once delivery

O evento será publicado ao menos uma vez. Pode haver duplicata se o poller falhar entre publicar e atualizar o flag:

```
 Poller              Broker             Outbox
 ┌──────┐          ┌────────┐        ┌────────────────┐
 │ pub  │─────────►│ ACK ✅  │        │ published=false│
 │      │          └────────┘        │                │
 │ ⚡💥  │ ◄── crash antes do UPDATE  │ ainda false!   │
 └──────┘                            └────────────────┘

 Próximo ciclo: poller re-seleciona o evento → DUPLICATA
 Solução: consumidores devem ser IDEMPOTENTES
```

### O que o padrão NÃO garante

- **Exactly-once delivery** — consumidores devem checar se já processaram o evento (tabela `processed_events` ou chave de idempotência).
- **Baixa latência** — intervalo de polling introduz delay. Para sub-segundo, usar CDC.

---

## Housekeeping

Eventos publicados devem ser limpos periodicamente para evitar crescimento indefinido:

```
 ┌──────────────────────────────────────────────────────┐
 │              CLEANUP (semanal)                        │
 │                                                       │
 │  outbox_events ──► outbox_events_archive              │
 │  (published=true     (histórico)                      │
 │   há mais de 7d)                                      │
 │                                                       │
 │  DELETE originais após arquivamento                   │
 └──────────────────────────────────────────────────────┘
```

---

## Visão Geral da Arquitetura

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                        SERVIÇO DE DOMÍNIO                       │
 │                                                                 │
 │  Controller ──► Use Case ──► Repository                         │
 │                    │                                            │
 │                    └──► OutboxRepository                        │
 └──────────┬──────────────────┬───────────────────────────────────┘
            │                  │
            ▼                  ▼
 ┌──────────────────┐  ┌──────────────────┐
 │ Tabela de        │  │ outbox_events    │◄──── OutboxPoller
 │ Negócio          │  │                  │      @Scheduled(5s)
 │ (manual,         │  │ published=false  │           │
 │  checklist...)   │  │ published=true   │           │
 └──────────────────┘  └──────────────────┘           │
                                                      ▼
                                              ┌──────────────┐
                                              │ SNS / SQS /  │
                                              │ Kafka         │
                                              └──────┬───────┘
                                     ┌───────────────┼───────────────┐
                                     ▼               ▼               ▼
                              ┌────────────┐  ┌────────────┐  ┌────────────┐
                              │ Consumidor │  │ Consumidor │  │ Consumidor │
                              │ RAG        │  │ TTS        │  │ Notificação│
                              └────────────┘  └────────────┘  └────────────┘
```
