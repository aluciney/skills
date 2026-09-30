---
name: criar-frontend
description: Cria um novo projeto frontend com Vite + React + TypeScript, instala libs padrão (axios, date-fns, lucide-react, motion, zod, react-hook-form, @hookform/resolvers, react-router, react-select), configura Tailwind CSS v4 via @tailwindcss/vite, ajusta o index.html (lang pt-BR e meta viewport com maximum-scale=1.0), configura o ESLint para aspas simples e sem ponto e vírgula com formatação automática ao salvar (.vscode/settings.json), monta a estrutura de pastas padrão (components, pages, hooks, services, @types, layouts, contexts, routes) e já deixa o roteamento pronto (App.tsx com RouterProvider, src/routes/index.ts com createBrowserRouter e uma página Home de boas-vindas no tema dark) e cria o CLAUDE.md do projeto apontando para esta skill. Use quando o usuário pedir para criar um novo projeto frontend/React/Vite do zero.
---

# Criar Projeto Frontend (Vite + React + TS + Tailwind)

Skill para inicializar um projeto frontend seguindo o padrão do usuário.

## Quando usar

Acione quando o usuário pedir para:
- "Criar um novo projeto frontend / React / Vite"
- "Iniciar um projeto novo com Tailwind"
- "Scaffold de projeto React TypeScript"

## Passos

### 1. Coletar o nome do projeto

Se o usuário não informou, pergunte o nome do projeto (em kebab-case). Não invente.

### 2. Criar o projeto Vite (não-interativo)

Use a forma não-interativa do `create vite` para evitar prompts:

```powershell
npm create vite@latest <nome-projeto> -- --template react-ts --eslint --no-interactive
```

As três flags são obrigatórias (o `create-vite` 9 mudou o comportamento padrão):

- `--template react-ts` — escolhe o template React + TypeScript.
- `--eslint` — sem ela o template React vem com **Oxlint** (`.oxlintrc.json`) em vez
  de ESLint, e o passo 7 não teria `eslint.config.js` para configurar.
- `--no-interactive` — em terminal TTY o CLI entra em modo interativo mesmo com o
  template informado e fica parado esperando resposta (framework, variante,
  instalar/rodar agora). Essa flag força o modo não-interativo.

Em seguida entre na pasta:

```powershell
Set-Location <nome-projeto>
```

### 3. Instalar dependências

Dependências de runtime:

```powershell
npm install axios date-fns lucide-react motion zod react-hook-form @hookform/resolvers react-router react-select
```

Tailwind v4 (com plugin Vite):

```powershell
npm install tailwindcss tailwind-merge @tailwindcss/vite
```

### 4. Configurar `vite.config.ts`

Substitua o conteúdo de `vite.config.ts` por:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

### 5. Importar Tailwind no CSS principal

Substitua **todo** o conteúdo de `src/index.css` (o CSS importado pelo `main.tsx`) por apenas:

```css
@import "tailwindcss";
```

Não mantenha o CSS de exemplo que o template do Vite deixa: regras globais como
`body { display: flex; place-items: center; }` e `:root { color-scheme: light dark; }`
conflitam com o layout em Tailwind e com o tema dark (ex.: quebram o `min-h-screen`
da página Home).

### 6. Ajustar o `index.html`

Edite o `index.html` na raiz do projeto (gerado pelo `create vite@latest`):

- Troque o idioma da tag `<html>` de `<html lang="en">` para `<html lang="pt-BR">`.
- Substitua a meta `viewport` por:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
```

### 7. Configurar ESLint (aspas simples, sem `;`) e formatação ao salvar

Instale o plugin de regras de estilo (o ESLint v9 não traz mais as regras de
formatação no core):

```powershell
npm install -D @stylistic/eslint-plugin
```

No `eslint.config.js` (flat config gerado pelo `create vite@latest`), registre o
plugin e adicione as regras de estilo **dentro do bloco de configuração dos
arquivos `.ts`/`.tsx`** (sem remover o que já existe — apenas acrescente o import,
a entrada em `plugins` e as duas regras). O bloco gerado hoje usa `extends` e não
tem chave `plugins`, então ela precisa ser criada:

```js
import stylistic from '@stylistic/eslint-plugin'

// ...na config dos arquivos .ts/.tsx (que já tem `files` e `extends`):
plugins: {
  '@stylistic': stylistic,
},
rules: {
  '@stylistic/quotes': ['error', 'single'],
  '@stylistic/semi': ['error', 'never'],
},
```

Para habilitar a **formatação automática ao salvar** aplicando essas correções,
crie `.vscode/settings.json` usando o ESLint como formatador (requer a extensão
`dbaeumer.vscode-eslint` instalada no VS Code):

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "editor.defaultFormatter": "dbaeumer.vscode-eslint"
}
```

Assim, ao salvar, o ESLint converte aspas duplas em simples e remove os `;` do
final das linhas automaticamente.

### 8. Criar a estrutura de pastas padrão

A partir da raiz padrão do `create vite@latest`, adicione (vazias com `.gitkeep`):

```
src/components/
src/pages/
src/hooks/
src/services/
src/@types/
src/layouts/
src/contexts/
src/routes/
```

Crie também o arquivo de tipagens globais:

```
src/@types/index.d.ts
```

### 9. Criar a página Home (boas-vindas, tema dark)

Crie `src/pages/Home/index.tsx` com a tela de boas-vindas abaixo. Ela usa Tailwind
(tema dark) e ícones do `lucide-react` (já instalado) e traz sugestões de pedidos
para o Claude:

```tsx
import { Sparkles, Code2, Palette, Rocket } from 'lucide-react'

export const Home: React.FC = () => {
  const suggestions = [
    {
      icon: Code2,
      title: 'Criar um componente',
      prompt: 'Crie um componente de tabela com paginação seguindo o padrão da skill',
    },
    {
      icon: Palette,
      title: 'Montar uma página',
      prompt: 'Crie uma página de login no tema dark com react-hook-form e zod',
    },
    {
      icon: Rocket,
      title: 'Configurar uma rota',
      prompt: 'Adicione uma rota /dashboard com um layout de admin',
    },
    {
      icon: Sparkles,
      title: 'Consumir uma API',
      prompt: 'Crie um service com axios e um hook para buscar usuários',
    },
  ]

  return (
    <main className="flex min-h-screen flex-col items-center justify-center bg-zinc-950 px-6 py-16 text-zinc-100">
      <div className="flex w-full max-w-3xl flex-col items-center text-center">
        <span className="inline-flex items-center gap-2 rounded-full border border-zinc-800 bg-zinc-900/60 px-4 py-1.5 text-sm text-zinc-400">
          <Sparkles className="h-4 w-4 text-violet-400" />
          Projeto criado com a skill criar-frontend
        </span>

        <h1 className="mt-6 bg-gradient-to-r from-violet-400 via-fuchsia-400 to-sky-400 bg-clip-text text-5xl font-bold tracking-tight text-transparent sm:text-6xl">
          Bem-vindo 👋
        </h1>

        <p className="mt-4 max-w-xl text-lg text-zinc-400">
          Seu projeto Vite + React + TypeScript + Tailwind está pronto. Peça ao
          Claude para construir a partir daqui — alguns exemplos:
        </p>

        <div className="mt-10 grid w-full grid-cols-1 gap-4 sm:grid-cols-2">
          {suggestions.map(({ icon: Icon, title, prompt }) => (
            <div
              key={title}
              className="group rounded-2xl border border-zinc-800 bg-zinc-900/50 p-5 text-left transition hover:border-violet-500/50 hover:bg-zinc-900"
            >
              <div className="flex items-center gap-3">
                <span className="flex h-10 w-10 items-center justify-center rounded-xl bg-violet-500/10 text-violet-400 transition group-hover:bg-violet-500/20">
                  <Icon className="h-5 w-5" />
                </span>
                <h3 className="font-semibold text-zinc-100">{title}</h3>
              </div>
              <p className="mt-3 text-sm text-zinc-400">"{prompt}"</p>
            </div>
          ))}
        </div>

        <p className="mt-12 text-sm text-zinc-600">
          Edite{" "}
          <code className="rounded bg-zinc-900 px-1.5 py-0.5 text-zinc-400">
            src/pages/Home/index.tsx
          </code>{" "}
          para começar.
        </p>
      </div>
    </main>
  )
}
```

> Exceção ao template padrão do passo 13: a Home já vem com JSX preenchido (não usa
> o `return ()` vazio), pois é a tela inicial pronta do projeto.

### 10. Configurar as rotas (`src/routes/index.ts`)

Crie `src/routes/index.ts` com o `createBrowserRouter`. Use a propriedade
`Component` (referência ao componente, sem JSX) para manter o arquivo como `.ts`:

```ts
import { createBrowserRouter } from 'react-router'
import { Home } from '../pages/Home'

export const router = createBrowserRouter([
  {
    path: '/',
    Component: Home,
  },
])
```

### 11. Configurar o `App.tsx` e o `main.tsx`

Substitua o conteúdo de `src/App.tsx` para usar o `RouterProvider`, seguindo o
padrão de export nomeado da skill:

```tsx
import { RouterProvider } from 'react-router'
import { router } from './routes'

export const App: React.FC = () => {
  return <RouterProvider router={router} />
}
```

Como o `App` agora é um export **nomeado**, ajuste o import em `src/main.tsx`
(o template do Vite usa export default):

```tsx
import { App } from './App.tsx'
```

Remova também o `import './App.css'` e qualquer CSS de exemplo que o template
do Vite tenha deixado, se não for usar.

### 12. Convenções de arquivos (a SEGUIR ao criar componentes/páginas/etc.)

Sempre que for criar artefatos novos depois disso, siga estes caminhos:

- `src/components/{{NomeComponente}}/index.tsx` — componente
- `src/components/{{NomeComponente}}/types.ts` — tipagens do componente
- `src/pages/{{NomePagina}}/index.tsx` — página
- `src/hooks/{{nomeHook}}.tsx` — hooks
- `src/services/{{nome}}.ts` — serviços (ex.: `api.ts` exportando o axios configurado)
- `src/@types/index.d.ts` — tipagens globais
- `src/layouts/{{Nome}}Layout.tsx` — layouts (ex.: `AdminLayout`, `LoginLayout`)
- `src/contexts/{{Nome}}Context.tsx` — contextos
- `src/routes/index.ts` — definição das rotas (`createBrowserRouter`); adicione novas rotas ao array com `Component:`

### 13. Padrão de componente / página

Sempre crie componentes e páginas neste formato:

```tsx
export const {{Nome}}: React.FC = () => {
  return ()
}
```

(O `return ()` é literal conforme padrão do usuário — o conteúdo JSX é preenchido em seguida.)

### 14. Criar o `CLAUDE.md` do projeto (obrigatório)

Crie `CLAUDE.md` na raiz do projeto (substitua `<nome-projeto>`) para que qualquer
alteração futura siga esta skill:

```markdown
# CLAUDE.md

Frontend **<nome-projeto>** — Vite + React + TypeScript + Tailwind CSS v4.

## Regra obrigatória

Este projeto foi criado pela skill **`criar-frontend`** e deve seguir o padrão dela.
**Toda vez que o usuário pedir qualquer alteração neste frontend** (novo componente, página,
hook, service, layout, context, rota, tipagem, dependência, refatoração etc.),
**invoque/consulte a skill `criar-frontend` antes de escrever código** e siga exatamente:

- padrão de componente/página: `export const Nome: React.FC = () => { ... }` (export nomeado, sem default);
- organização de pastas e nomenclatura: `components/<Nome>/index.tsx` + `types.ts`, `pages/<Nome>/index.tsx`,
  `hooks/<nomeHook>.tsx`, `services/<nome>.ts`, `layouts/<Nome>Layout.tsx`, `contexts/<Nome>Context.tsx`,
  `@types/index.d.ts`, rotas em `routes/index.ts` (`createBrowserRouter` com `Component:`);
- estilo com Tailwind (tema dark), ícones `lucide-react`, formulários com `react-hook-form` + `zod`, HTTP com `axios`;
- código com aspas simples e sem `;` (ESLint `@stylistic`);
- não adicionar bibliotecas além das padrão sem o usuário pedir.

Em caso de dúvida sobre o padrão, a skill é a fonte de verdade — não invente outro.

## Comandos

- `npm run dev` — sobe em modo desenvolvimento
- `npm run build` — build de produção
- `npm run lint` — lint (ESLint)
```

## Verificação final

Após terminar, rode uma vez:

```powershell
npm run dev
```

Confirme que o servidor sobe sem erro e reporte a URL ao usuário. Caso haja erro de tipos/lint, corrija antes de encerrar.

## Notas

- Use **PowerShell** (ambiente Windows). Não use sintaxe bash (`&&`, `$VAR`, `/dev/null`).
- Para encadear comandos use `;` ou `if ($?) { ... }`.
- Não rode `npm create vite@latest` de forma interativa — sempre passe `<nome> -- --template react-ts --eslint --no-interactive`.
- Não adicione bibliotecas além das listadas a menos que o usuário peça.
