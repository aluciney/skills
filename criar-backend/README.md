# criar-backend — Skill do Claude Code

Skill para **criar e manter backends Node.js/TypeScript** seguindo uma arquitetura padrão
(Fastify 5 + fastify-type-provider-zod + Zod 4 + Knex + oracledb, testes com Jest, lint/format
com Biome). Faz bootstrap de projeto do zero e gera módulos/rotas/plugins no padrão.

> A raiz deste repositório é o conteúdo da skill: o `SKILL.md` fica na raiz e os modelos em `templates/`.

## Instalar (global — disponível em todos os projetos)

```powershell
git clone git@gitlab.amazonasenergia.com:skills/criar-backend.git "$env:USERPROFILE\.claude\skills\criar-backend"
```

## Instalar (apenas em um projeto)

```powershell
git clone git@gitlab.amazonasenergia.com:skills/criar-backend.git ".\.claude\skills\criar-backend"
```

> O resultado precisa ser `.../.claude/skills/criar-backend/SKILL.md`.

## Atualizar

```powershell
cd "$env:USERPROFILE\.claude\skills\criar-backend"; git pull
```

## Usar

No Claude Code, invoque com `/criar-backend` ou peça em linguagem natural, por exemplo:

- "Cria um backend novo chamado `loja-api`"
- "Cria o módulo `usuario` com rota autenticada GET /perfil e POST /criar"
- "Cria um plugin de auditoria"

## Estrutura

```
SKILL.md
templates/
  base/      # bootstrap de um backend do zero (config + src/ completo)
  module/    # os 6 arquivos de um módulo
  plugin/    # plugin Fastify (fastify-plugin)
```
