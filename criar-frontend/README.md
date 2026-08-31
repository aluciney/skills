# criar-frontend

Skill do Claude Code que cria um novo projeto frontend já com a stack e a estrutura de pastas padrão configuradas.

## O que a skill faz

Quando acionada, ela executa em sequência:

1. Cria um projeto **Vite + React + TypeScript** de forma não-interativa
   (`npm create vite@latest <nome> -- --template react-ts --eslint --no-interactive`).
   O `--eslint` é necessário porque o `create-vite` 9 passou a usar **Oxlint** por
   padrão nos templates React, e o `--no-interactive` evita que o CLI abra os
   prompts (e fique travado) mesmo com o template já informado.
2. Instala as libs de runtime:
   `axios`, `date-fns`, `lucide-react`, `motion`, `zod`,
   `react-hook-form`, `@hookform/resolvers`, `react-router`, `react-select`.
3. Instala o **Tailwind CSS v4**:
   `tailwindcss`, `tailwind-merge`, `@tailwindcss/vite`.
4. Sobrescreve `vite.config.ts` registrando os plugins `react()` e `tailwindcss()`.
5. Substitui todo o conteúdo de `src/index.css` por `@import "tailwindcss";` (removendo o CSS de exemplo do Vite, que conflita com o tema dark).
6. Ajusta o `index.html`: troca `<html lang="en">` por `<html lang="pt-BR">` e a meta viewport por `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />`.
7. Configura o **ESLint** com `@stylistic/eslint-plugin` para usar **aspas simples** (`@stylistic/quotes`) e **sem ponto e vírgula** (`@stylistic/semi`), e cria `.vscode/settings.json` habilitando a **formatação automática ao salvar** via ESLint (`source.fixAll.eslint`).
8. Cria a estrutura de pastas padrão:
   ```
   src/components/
   src/pages/
   src/hooks/
   src/services/
   src/@types/index.d.ts
   src/layouts/
   src/contexts/
   src/routes/
   ```
9. Deixa o **roteamento pronto**:
   - `src/pages/Home/index.tsx` — uma tela de boas-vindas no tema dark com sugestões de pedidos para o Claude.
   - `src/routes/index.ts` — `createBrowserRouter` (importado de `react-router`) com a rota `/` apontando para a `Home`.
   - `src/App.tsx` — sobrescrito com `<RouterProvider router={router} />` (export nomeado), ajustando o import em `src/main.tsx`.
10. Roda `npm run dev` para validar.

A skill também documenta o padrão de caminhos e o template de componente/página:

```tsx
export const {{Nome}}: React.FC = () => {
  return ()
}
```

> A página `Home` é uma exceção: já vem com JSX preenchido por ser a tela inicial pronta do projeto.

## Como acionar

Basta pedir no Claude Code algo como:

- "crie um novo projeto frontend chamado **meu-app**"
- "scaffold de um projeto React + Vite + Tailwind chamado **dashboard-admin**"
- "inicia um projeto novo do zero"

O Claude Code detecta a skill pelo `description` no frontmatter do `SKILL.md` e a invoca automaticamente.

## Instalação

A skill é um diretório contendo um `SKILL.md`. Existem dois locais possíveis:

| Escopo | Caminho | Quando usar |
| --- | --- | --- |
| **Local** (por projeto) | `<projeto>/.claude/skills/criar-frontend/` | Disponível só dentro daquele projeto. Versionada no repo do projeto. |
| **Global** (usuário) | `~/.claude/skills/criar-frontend/` (Windows: `%USERPROFILE%\.claude\skills\criar-frontend\`) | Disponível em qualquer projeto da sua máquina. |

### Instalação local (por projeto) via GitLab

Dentro do projeto onde quer usar a skill:

```powershell
# Garante que a pasta existe
New-Item -ItemType Directory -Force .\.claude\skills | Out-Null

# Clona a skill diretamente para dentro de .claude/skills/
git clone git@gitlab.amazonasenergia.com:skills/criar-frontend.git .\.claude\skills\criar-frontend
```

Para versionar a skill junto do projeto, basta commitar o diretório `.claude/skills/criar-frontend/` no repositório.

Se preferir mantê-la atrelada ao repositório original (em vez de copiar os arquivos), use um **submódulo**:

```powershell
git submodule add git@gitlab.amazonasenergia.com:skills/criar-frontend.git .claude/skills/criar-frontend
git commit -m "chore: add criar-frontend skill as submodule"
```

Para atualizar depois:

```powershell
git submodule update --remote .claude/skills/criar-frontend
```

### Instalação global (em toda a máquina) via GitLab

No Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
git clone git@gitlab.amazonasenergia.com:skills/criar-frontend.git "$env:USERPROFILE\.claude\skills\criar-frontend"
```

No Linux/macOS (bash):

```bash
mkdir -p ~/.claude/skills
git clone git@gitlab.amazonasenergia.com:skills/criar-frontend.git ~/.claude/skills/criar-frontend
```

Depois disso a skill fica disponível em qualquer diretório onde você abrir o Claude Code.

Para atualizar a versão global:

```powershell
git -C "$env:USERPROFILE\.claude\skills\criar-frontend" pull
```

### Verificar se a skill foi carregada

Abra o Claude Code no diretório e rode:

```
/help
```

Ou simplesmente peça "crie um novo projeto frontend chamado teste" — se a skill estiver instalada, ela será invocada via `Skill`.

## Publicando no GitLab

A partir deste diretório (`.claude/skills/criar-frontend/`):

```powershell
git init
git add SKILL.md README.md
git commit -m "feat: initial skill"
git branch -M main
git remote add origin git@gitlab.amazonasenergia.com:skills/criar-frontend.git
git push -u origin main
```

> Dica: o repositório no GitLab deve conter **apenas** o conteúdo da pasta da skill
> (`SKILL.md`, `README.md` e quaisquer arquivos auxiliares) — não o `.claude/skills/` em volta.
> Assim o `git clone <repo> <destino>/criar-frontend` posiciona os arquivos no lugar certo.

## Estrutura do repositório da skill

```
criar-frontend/
├── SKILL.md     # Instruções que o Claude Code lê para executar a skill
└── README.md    # Este arquivo (documentação humana)
```

## Customizar

Edite `SKILL.md` para:

- Alterar a lista de dependências instaladas.
- Mudar o template do `vite.config.ts`.
- Ajustar a estrutura de pastas padrão.
- Trocar o padrão de componente/página.

Após editar e commitar, atualize as instalações com `git pull` (global) ou `git submodule update --remote` (submódulo).
