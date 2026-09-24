---
name: criar-backend
description: >-
  Cria backends Node.js/TypeScript do zero e os mantém, seguindo uma arquitetura padrão
  (Fastify 5 + fastify-type-provider-zod + Zod 4 + Knex + oracledb, testes com Jest, lint/format
  com Biome). Use quando o usuário pedir para iniciar/bootstrap um novo backend/API/projeto, ou
  para gerar/adicionar um módulo, rota, endpoint, controller, service, dao, schema, teste (spec)
  ou um plugin Fastify dentro desse padrão. Garante organização de pastas, nomenclatura de
  arquivos, uso dos tipos globais de @types/moduleSchema.d.ts, preHandler [app.auth] em rotas
  autenticadas, registro em router.ts/app.ts e instalação de libs sempre na versão mais atual
  (exceto oracledb, sempre fixado em 5.4.0, e typescript, fixado em ^6).
---

# Criar Backend — bootstrap + geração de módulos/plugins

Esta skill cria e mantém backends seguindo uma arquitetura padrão. Os templates em `templates/`
são a fonte de verdade — sempre parta deles e preencha os placeholders. Há dois cenários:

1. **Bootstrap** — montar um backend novo do zero (`templates/base/`).
2. **Módulo/Plugin** — adicionar partes a um backend já existente (`templates/module/`, `templates/plugin/`).

## Stack e regras de dependências

Stack: **Fastify 5 + fastify-type-provider-zod + Zod 4 + Knex + oracledb**, TypeScript, Jest, Biome.

Ao adicionar qualquer biblioteca:

- Instale **sempre a versão mais atual**: `npm install <lib>@latest` (use `-D` para devDependencies).
- **Exceções:** `oracledb` **sempre fixado em `5.4.0`** (sem `^`): `npm install oracledb@5.4.0`,
  e confirme que o `package.json` mostra `"oracledb": "5.4.0"`.
- `typescript` **fixado em `^6`** (`npm install -D typescript@^6`): o `ts-jest` (29.x) só aceita `typescript >=4.3 <7`.
  Com TS 7 o npm dá `ERESOLVE`. Suba para a última só quando o `ts-jest` passar a suportá-la.
- Não fixe as outras libs com `=`; mantenha o range `^` que o `npm install` cria.
- Os templates `package.json` **não trazem versões** de propósito — as deps são instaladas via npm
  para sempre pegar a última.

## Placeholders dos templates

| Placeholder | Significado | Exemplo |
|---|---|---|
| `{{project}}` | nome do pacote (kebab-case) | `tas-backend` |
| `{{Project}}` | título legível (Swagger/README) | `TAS` |
| `{{db}}` | alias do banco (minúsculo) | `ajuri` |
| `{{DB}}` | sufixo do alias nas envs (MAIÚSCULO) | `AJURI` |
| `{{module}}` | nome do módulo (minúsculo, português) | `autenticacao` |
| `{{Module}}` | módulo em PascalCase | `Autenticacao` |
| `{{action}}` | ação/operação | `login` |
| `{{Action}}` | ação em PascalCase | `Login` |
| `{{name}}` | nome do plugin (minúsculo) | `auth` |

## Convenções de código (siga exatamente)

- **Indentação com TAB**, **sem ponto e vírgula**, **aspas simples**, `lineWidth` 200 (config Biome).
- Módulo `{{module}}`: nome em **minúsculo, em português**.
- **Classes** em PascalCase: `{{Module}}Controller`, `{{Module}}Service`.
- **DAO** e **Schema**: objetos literais exportados como `const` (`{{Module}}DAO`, `{{Module}}Schema`).
- **Routes**: arrow function `export const {{module}}Routes = (app: FastifyTypedInstance) => { ... }`.
- **Aliases obrigatórios em todo import** (nunca caminhos relativos como `./` ou `../`):
  - `@/*` → arquivos dentro de `src/` (ex.: `@/types`, `@/modules/{{module}}/{{module}}.service`).
  - `@root/*` → arquivos na raiz do projeto (ex.: `@root/knexfile`).
  - Vale também para arquivos da raiz que importam de `src/` (ex.: `knexfile.ts` usa `@/env`, não `./src/env`).
  - Imports de pacotes/node_modules continuam pelo nome do pacote (ex.: `from 'zod'`).

### Conexão com o banco (trx) — uma instância por request, vinda do controller

**Não existe pool compartilhado entre requests.** Cada request cria a própria instância do Knex e a
destrói no fim. Quem faz isso é o `database.plugin`, via `decorateRequest('trx')` + hook
`onRequest` (cria com `createKnex{{DB}}()`) + hooks `onResponse`/`onRequestAbort` (chamam `destroy()`).
O `knexfile.ts` usa `pool: { min: 0, max: 1 }` — o Knex sempre tem pool (tarn), então "uma conexão"
se expressa assim, não removendo o `pool`.

Sempre que um **service** precisar usar um **DAO**, a conexão deve ser **enviada pelo controller** e
repassada até o DAO. **Por padrão use sempre `request.trx`** (nunca `request.server.trx`, que não
existe mais — a conexão é do request, não da instância do servidor):

- **Controller**: passa `request.trx` como **primeiro argumento** ao service →
  `{{module}}Service.{{action}}(request.trx, request.body)`.
- **Service**: recebe `trx: Knex` como primeiro parâmetro e o repassa ao DAO →
  `{{Module}}DAO.{{action}}({ trx })`.
- **DAO**: todo método recebe `props: IncludeTRX<{...}>` (tipo global de `@types/transaction.d.ts`,
  que soma `{ trx: Knex }` às props do método) e usa `props.trx` para as queries. Sem outras props,
  use `props: IncludeTRX`.
- **Desestruturação de props**: quando as props tiverem **poucos atributos (até 5, contando o `trx`)**,
  sempre desestruture na primeira linha da função, com o **`trx` sempre por último**. Vale para DAO,
  service e qualquer função que receba `props`. Com mais atributos, mantenha `props.<campo>`, mas
  **sempre desestruture o `trx`** (`const { trx } = props`) e use `trx` em vez de `props.trx`.
  ```ts
  buscar: async (props: IncludeTRX<{ id_cliente: string }>) => {
  	const { id_cliente, trx } = props
  	return trx('clientes').where({ id_cliente }).first()
  },
  ```
- Tipo: use `Knex` (não `Knex.Transaction`), pois `request.trx` é uma instância `Knex`.
- **Nunca** chame `commit()`/`rollback()`/`destroy()` no controller: `commit`/`rollback` não existem
  numa instância `Knex` (só em `Knex.Transaction`) e o `destroy()` é do plugin. Se a ação precisar de
  transação, use `request.trx.transaction(async (t) => { ... })` no service — ele já faz commit no
  sucesso e rollback no erro.
- Como `max: 1`, queries em paralelo (`Promise.all`) na mesma instância **serializam**. Se precisar de
  paralelismo real dentro de um request, aumente o `max` no `knexfile.ts`.
- Rotas **WebSocket** não recebem `request.trx` (o reply é sequestrado e o `onResponse` não dispara).
  Nelas, chame `createKnex{{DB}}()` e faça `destroy()` no `close` do socket.

### WebSocket — `wsHub` já disponível em `libs/ws-hub.ts`

O bootstrap já instala e registra `@fastify/websocket` (no `app.ts` → `registerPlugins()`) e inclui a
lib `src/libs/ws-hub.ts`, que exporta o singleton **`wsHub`** — um hub de conexões agrupadas por canal.
Não é preciso criar nada: se o módulo precisar de WebSocket, basta importar `wsHub` e usá-lo.

- API: `wsHub.subscribe(channel, socket)`, `wsHub.unsubscribe(channel, socket)`,
  `wsHub.publish(channel, payload)` (serializa com `JSON.stringify` e envia só a sockets `OPEN`) e
  `wsHub.stats()` (`{ channels, clients }`).
- Padrão: use o **id do recurso como canal** (canal privado) para notificar eventos de atualização.
- Numa rota WebSocket, inscreva o socket ao conectar e cancele a inscrição no `close`; em qualquer
  ponto (controller/service) chame `wsHub.publish(id, payload)` para emitir aos inscritos daquele canal.

```ts
import { wsHub } from '@/libs/ws-hub'

app.get('/:id/eventos', { websocket: true }, (socket, request) => {
	const { id } = request.params as { id: string }
	wsHub.subscribe(id, socket)
	socket.on('close', () => wsHub.unsubscribe(id, socket))
})
```

### Testes (spec) — sempre com `jest.mock` no DAO

No `{{module}}.spec.ts`, sempre que o módulo usar um DAO, faça **`jest.mock` do arquivo do DAO**
para que os testes **não dependam de um banco real**:

- `jest.mock('@/modules/{{module}}/{{module}}.dao', () => ({ {{Module}}DAO: { {{action}}: jest.fn() } }))`
  (a chamada é içada/hoisted para o topo pelo Jest).
- Importe `jest` de `@jest/globals` e defina o retorno simulado com
  `jest.mocked({{Module}}DAO.{{action}}).mockResolvedValue({ ... })` dentro do `it`.
- Para cada método de DAO usado pelo módulo, adicione um `jest.fn()` correspondente no factory do mock.

## Organização de pastas (padrão obrigatório)

```
{{project}}/
  package.json  tsconfig.json  biome.json  jest.config.ts  tsup.config.ts  knexfile.ts  .gitignore
  src/
    server.ts            # ponto de entrada (bootstrap da App)
    app.ts               # classe App: compilers, segurança, plugins, hooks, swagger, rotas
    router.ts            # registra os *Routes de cada módulo sob /api
    env.ts               # validação de env com Zod
    types.ts             # FastifyTypedInstance, TypedRequest, TypedReply
    @types/
      index.d.ts         # augmentations (FastifyInstance.auth, FastifyRequest.trx, @fastify/jwt)
      moduleSchema.d.ts  # tipos globais ModuleSchema + InferModuleSchema
      transaction.d.ts   # tipo global IncludeTRX<T> (injeta { trx: Knex } nas props)
    libs/
      knex.ts            # createKnex{{DB}}(): factory de instância do Knex + knex-paginate
      sema.ts            # InstrumentedSema (semáforo de concorrência)
      ws-hub.ts          # wsHub: hub de WebSocket por canal (subscribe/unsubscribe/publish)
    plugins/
      auth.plugin.ts
      database.plugin.ts
      error-handler.plugin.ts
    modules/
      {{module}}/
        {{module}}.routes.ts
        {{module}}.schema.ts
        {{module}}.controller.ts
        {{module}}.service.ts
        {{module}}.dao.ts
        {{module}}.spec.ts
```

## Cenário 1 — Bootstrap de um backend novo

1. Pergunte (ou deduza): `{{project}}`, `{{Project}}` e o alias de banco `{{db}}`/`{{DB}}`.
2. Crie todos os arquivos a partir de `templates/base/` (mantendo a árvore acima), substituindo placeholders.
3. Instale as dependências (sempre a última versão; oracledb e typescript fixos):
   ```
   npm install fastify @fastify/cookie @fastify/cors @fastify/helmet @fastify/jwt \
     @fastify/rate-limit @fastify/sensible @fastify/swagger @fastify/swagger-ui \
     @fastify/websocket fastify-plugin fastify-type-provider-zod \
     zod knex knex-paginate async-sema date-fns dotenv
   npm install oracledb@5.4.0 -E
   npm install -D typescript@^6 tsx ts-node tsup @types/node @biomejs/biome \
     jest ts-jest @types/jest
   ```
   O `tsup` é só ferramenta de build: fica sempre em `devDependencies`, nunca em `dependencies`.
4. Crie um `.env` a partir de `templates/base/env.example.tmpl`.
5. Valide com `npx tsc --noEmit`. Suba com `npm run dev`.

## Cenário 2 — Criar um MÓDULO novo

1. Pergunte/deduza: nome do módulo, ações (método HTTP + path), se cada rota é **autenticada**,
   e os campos de Body/Query/Params/Response.
2. Crie `src/modules/{{module}}/` com os 6 arquivos a partir de `templates/module/`:
   `{{module}}.schema.ts`, `{{module}}.routes.ts`, `{{module}}.controller.ts`,
   `{{module}}.service.ts`, `{{module}}.dao.ts`, `{{module}}.spec.ts`.
3. **Registre no `src/router.ts`**: import + `app.register({{module}}Routes, { prefix: '/{{module}}' })`.
4. Valide com `npx tsc --noEmit`.

### Schema (`{{module}}.schema.ts`) — sempre via @types/moduleSchema.d.ts

- O objeto `{{Module}}Schema` termina com `satisfies ModuleSchema`.
- Cada ação tem `Body?`, `Query?`, `Params?` (opcionais) e `Response` (obrigatório, com chaves de
  status HTTP: `200 | 201 | 400 | 401 | 404 | 500`).
- Exporte `{{Module}}SchemaType` derivando cada ação com `InferModuleSchema<...>`.
- `ModuleSchema` e `InferModuleSchema` são **globais** (não precisam de import).

### Rotas autenticadas — preHandler [app.auth]

Se a rota for **autenticada**, adicione `preHandler: [app.auth]` nas opções da rota, **antes** do
`schema`, e inclua `security: [{ bearerAuth: [] }]` dentro de `schema` (reflete no Swagger):

```ts
app.get('/perfil', {
	preHandler: [app.auth],
	schema: {
		tags: ['{{module}}'],
		summary: 'Dados do usuário autenticado',
		security: [{ bearerAuth: [] }],
		response: {{Module}}Schema.perfil.Response,
	},
	handler: {{module}}Controller.perfil.bind({{module}}Controller),
})
```

Rotas públicas (ex.: login) **não** levam `preHandler`.

## Cenário 3 — Criar um PLUGIN novo

1. Crie `src/plugins/{{name}}.plugin.ts` a partir de `templates/plugin/plugin.plugin.ts.tmpl`.
2. Use sempre `fastify-plugin` (`fp`): `export const {{name}}Plugin = fp(async (app) => { ... })`.
3. Se o plugin **decora**, adicione a tipagem em `src/@types/index.d.ts` (`declare module 'fastify'`):
   `app.decorate('algo', ...)` → `interface FastifyInstance`; `app.decorateRequest('algo', ...)` →
   `interface FastifyRequest`. Para valores por request, decore com `null` e preencha num hook
   `onRequest` (Fastify 5 não copia objetos entre requests) — ver `database.plugin.ts`.
4. **Registre no `src/app.ts`** dentro de `registerPlugins()`, na ordem correta de dependências.

## Checklist final (sempre)

- [ ] Pastas e arquivos no padrão (`{{module}}.{tipo}.ts`, `{{name}}.plugin.ts`).
- [ ] Schema usa `satisfies ModuleSchema` + `InferModuleSchema` (tipos globais).
- [ ] DAO tipa as props com `IncludeTRX<...>` (tipo global).
- [ ] Props com até 5 atributos são desestruturadas (`const { campo, trx } = props`), com `trx` por último; com mais, só o `trx` (`const { trx } = props`).
- [ ] Rotas autenticadas têm `preHandler: [app.auth]` (+ `security` no swagger).
- [ ] Controller passa `request.trx` ao service; service repassa `{ trx }` ao DAO.
- [ ] Nenhum `commit()`/`rollback()`/`destroy()` no controller (ciclo de vida é do `database.plugin`).
- [ ] Spec faz `jest.mock` de cada DAO usado (testes não dependem de banco real).
- [ ] Módulo registrado no `router.ts`; plugin registrado no `app.ts`.
- [ ] Libs novas na última versão; `oracledb` fixado em `5.4.0`; `typescript` em `^6`; `tsup` em devDependencies.
- [ ] Todo import usa alias (`@/` para `src/`, `@root/` para a raiz) — sem caminhos relativos.
- [ ] Indentação com TAB, sem `;`, aspas simples.
