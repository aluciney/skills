# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este repositório

Este repositório **é uma skill do Claude Code** (`criar-backend`), não uma aplicação. Sua função é
criar e manter backends Node.js/TypeScript a partir de templates. A "fonte de verdade" são:

- **`SKILL.md`** — instruções, convenções e checklists que orientam a geração de código.
- **`templates/`** — arquivos `.tmpl` com placeholders que viram os arquivos reais nos projetos gerados.

Não há build/lint/test **deste** repositório. Editar a skill = editar `SKILL.md` e os `.tmpl`. Os comandos
de `npm` abaixo pertencem aos **backends gerados**, não a este repo.

## Como trabalhar nos templates

- Os `.tmpl` são copiados e têm placeholders substituídos. Mantenha-os **sintaticamente válidos** no
  padrão de código (TAB, sem `;`, aspas simples, `lineWidth` 200) — eles devem passar `tsc` após o preenchimento.
- Placeholders (definidos em `SKILL.md`): `{{project}}`/`{{Project}}` (pacote/título), `{{db}}`/`{{DB}}`
  (alias do banco minúsculo/MAIÚSCULO), `{{module}}`/`{{Module}}`, `{{action}}`/`{{Action}}`, `{{name}}` (plugin).
- Ao mudar uma convenção, atualize **os dois lados**: o(s) `.tmpl` afetado(s) **e** a descrição/checklist
  correspondente em `SKILL.md`. Eles precisam ficar coerentes — `SKILL.md` é o que a skill realmente segue.
- `templates/base/` = bootstrap de um projeto do zero. `templates/module/` = os 6 arquivos de um módulo.
  `templates/plugin/` = um plugin Fastify.

## Arquitetura dos backends gerados

Stack: **Fastify 5 + fastify-type-provider-zod + Zod 4 + Knex + oracledb**, TypeScript, Jest, Biome.
Regra de deps: sempre `@latest`, **exceto `oracledb` fixado em `5.4.0`** (sem `^`).

**Fluxo de uma requisição** (camadas, do externo ao banco):

```
routes (Zod schema + preHandler) → controller → service → DAO → Knex
```

- **`app.ts`** — classe `App` que monta o servidor numa ordem fixa: `registerCompilers` (Zod) →
  `registerSecurity` (helmet, rate-limit) → `registerPlugins` (sensible, error-handler, cookie, jwt, cors, auth)
  → `registerHooks` (semáforo de concorrência via `InstrumentedSema`, libera no `finish`/`close`/`error` do reply)
  → `registerSwagger` → `registerRoutes`. Expõe `ready()`, `listen()`, `close()`.
- **`router.ts`** — registra os `*Routes` de cada módulo sob o prefixo `/api`.
- Cada módulo registra suas rotas com um prefixo próprio: `app.register({{module}}Routes, { prefix: '/{{module}}' })`.

**Tipagem dirigida por schema (central):**

- `src/@types/moduleSchema.d.ts` define os tipos **globais** `ModuleSchema` e `InferModuleSchema` (sem import).
- Cada `{{module}}.schema.ts` exporta um objeto `{{Module}}Schema` terminando em `satisfies ModuleSchema`,
  com `Body?`/`Query?`/`Params?` + `Response` (chaveado por status HTTP), e deriva `{{Module}}SchemaType`
  via `InferModuleSchema`.
- `src/types.ts` define `FastifyTypedInstance`, `TypedRequest<T>`, `TypedReply<T>` (Fastify + `ZodTypeProvider`).
  Controllers usam `TypedRequest`/`TypedReply` tipados pela ação do schema.

**Conexão com o banco (regra de fluxo):** a conexão sempre **desce pelo controller**. O controller passa
`request.server.trx` (instância `Knex` decorada pelo `database.plugin`) como **primeiro argumento** ao service;
o service repassa `{ trx }` ao DAO; todo método de DAO recebe `props: { trx: Knex }`. Use o tipo `Knex`
(não `Knex.Transaction`), pois `request.server.trx` é uma instância `Knex`.

**Autenticação:** rotas protegidas levam `preHandler: [app.auth]` (decorado pelo `auth.plugin`) **antes** do
`schema`, e `security: [{ bearerAuth: [] }]` dentro do `schema` (reflete no Swagger). Rotas públicas não têm `preHandler`.

**Plugins:** sempre via `fastify-plugin` (`fp`). Se decorarem a instância, a tipagem vai em
`src/@types/index.d.ts` (`declare module 'fastify'` → `interface FastifyInstance`), e o registro em
`app.ts` → `registerPlugins()` na ordem de dependência.

**Testes:** o `{{module}}.spec.ts` sobe a `App` real e usa `app.server.inject(...)`, mas faz **`jest.mock` do
DAO** para não depender de banco — define retornos com `jest.mocked({{Module}}DAO.{{action}}).mockResolvedValue(...)`.

## Comandos (dos backends gerados, não deste repo)

- `npm run dev` — `tsx watch src/server.ts`
- `npm run build` — `tsup`
- `npm start` — `node dist/server.js`
- `npm test` — `jest` (rodar um arquivo: `npx jest src/modules/<module>/<module>.spec.ts`;
  por nome: `npx jest -t "<trecho do nome do teste>"`)
- `npx tsc --noEmit` — checagem de tipos (rodar após gerar/editar arquivos)

Convenções de código: indentação com **TAB**, **sem `;`**, **aspas simples**, `lineWidth` 200 (Biome).
Aliases: `@/*` → `src/*`, `@root/*` → raiz do projeto.
