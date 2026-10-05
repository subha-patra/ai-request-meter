# AI Request Meter

[![npm version](https://img.shields.io/npm/v/ai-request-meter?cacheSeconds=60)](https://www.npmjs.com/package/ai-request-meter)
[![npm downloads](https://img.shields.io/npm/dt/ai-request-meter?cacheSeconds=60)](https://www.npmjs.com/package/ai-request-meter)
[![GitHub issues](https://img.shields.io/github/issues/subha-patra/ai-request-meter?cacheSeconds=60)](https://github.com/subha-patra/ai-request-meter/issues)
[![GitHub stars](https://img.shields.io/github/stars/subha-patra/ai-request-meter?style=social&cacheSeconds=60)](https://github.com/subha-patra/ai-request-meter)
[![GitHub license](https://img.shields.io/github/license/subha-patra/ai-request-meter?cacheSeconds=60)](https://github.com/subha-patra/ai-request-meter/blob/main/LICENSE)

`ai-request-meter` is a server-first TypeScript SDK for tracking AI calls made through your application. It records token usage, estimated cost, latency, model, status, and local heuristic scores after each OpenAI-compatible response.

Current version: `1.0.0`

NPM package: [ai-request-meter](https://www.npmjs.com/package/ai-request-meter)

Repository: [GitHub](https://github.com/subha-patra/ai-request-meter)

Live demo and docs: [Demo](https://subha-patra.github.io/ai-request-meter/)

---

## Install

```bash
npm install ai-request-meter
```

`pg` is optional at package level. Install it only when you use `postgresLogger()`:

```bash
npm install pg
```

---

## Basic Usage

```ts
import { createAI, openaiProvider, postgresLogger } from 'ai-request-meter';

const ai = createAI({
  provider: openaiProvider({
    apiKey: process.env.OPENAI_API_KEY
  }),
  logger: postgresLogger({
    connectionString: process.env.DATABASE_URL
  })
});

const result = await ai.generate({
  taskName: 'customer_support_reply',
  model: 'gpt-4o-mini',
  messages: [
    { role: 'user', content: 'Write reply for this customer' }
  ]
});

console.log(result.usage, result.estimatedCost, result.scores);
```

Tracking works automatically when developers call AI through `ai.generate()`. It does not secretly intercept direct SDK calls made outside `ai-request-meter`.

---

## PostgreSQL Migration

```sql
create table if not exists ai_request_meter_logs (
  id text primary key,
  task_name text not null,
  provider text not null,
  model text not null,
  input_tokens integer not null default 0,
  output_tokens integer not null default 0,
  total_tokens integer not null default 0,
  estimated_cost numeric not null default 0,
  latency_ms integer not null default 0,
  status text not null,
  error_message text,
  quality_score integer not null default 0,
  content_score integer not null default 0,
  readiness_score integer not null default 0,
  scores jsonb not null default '{}'::jsonb,
  metadata jsonb not null default '{}'::jsonb,
  prompt_text text,
  response_text text,
  created_at timestamptz not null
);
```

---

## Privacy Defaults

By default, prompt and response text are opt-in. `ai-request-meter` stores metadata, token usage, estimated cost, model, provider, status, latency, and scores.

```ts
const ai = createAI({
  provider,
  logger,
  captureText: false
});
```

Use `captureText: true` only when your application policy allows prompt and response storage.

---

## API

```ts
export {
  createAI,
  openaiProvider,
  postgresLogger,
  consoleLogger,
  noopLogger,
  estimateCost,
  scoreQuality,
  scoreContent,
  scoreReadiness
};
```

- `createAI(config)` creates a tracked client.
- `ai.generate(options)` runs one text chat-completion style request.
- `openaiProvider(options)` supports OpenAI-compatible `/chat/completions` APIs.
- `postgresLogger(options)` writes records into PostgreSQL.
- `consoleLogger()` logs records locally while testing.
- `noopLogger()` disables logging without changing call sites.
- `estimateCost()` estimates token cost using built-in or custom model pricing.
- `scoreQuality()`, `scoreContent()`, and `scoreReadiness()` run local deterministic heuristics.

The built-in `gpt-4o-mini` default pricing follows OpenAI's published token pricing at implementation time. For production billing accuracy, pass your own pricing map when your provider or contract differs.

---

## Logger Error Behavior

Logging failures do not break AI responses by default.

```ts
const ai = createAI({
  provider,
  logger,
  failOnLoggerError: true
});
```

Use `failOnLoggerError: true` for strict pipelines where a missing log should fail the request.

---

## Scope

v1 supports basic text chat-completion style calls through OpenAI-compatible APIs. Streaming, tool calls, embeddings, image inputs, MongoDB, SQLite, and hosted dashboards are planned as later additions.


## 📄 License

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
