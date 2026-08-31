# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A standalone **Claude Code skill** named `criar-frontend`. When invoked, it scaffolds a new frontend project (Vite + React + TypeScript + Tailwind v4 + a fixed dependency set) following the folder/code conventions defined in [SKILL.md](SKILL.md).

The repo contains only two source files:

- [SKILL.md](SKILL.md) — instructions the Claude Code runtime reads to execute the skill. The YAML frontmatter (`name`, `description`) controls when the runtime triggers it; editing the description changes the trigger surface.
- [README.md](README.md) — human-facing docs covering install (local per-project vs. global per-user) and publishing flow via GitLab.

There is no build, lint, or test step — the deliverable is markdown.

## Editing rule: keep SKILL.md and README.md in sync

User-facing changes (adding/removing a dependency, changing the Vite config, altering the folder layout, changing the component template) must land in **both** files or they drift:

- `SKILL.md` — update the `description:` frontmatter *and* the relevant step in the body.
- `README.md` — update the corresponding bullet under "O que a skill faz" (and any other place that mentions it).

The `description:` matters: the Claude Code runtime uses it (not the body) to decide whether to invoke the skill. A capability missing from the description may not get triggered even if the body mentions it.

## Install layout the skill must support

The README documents two install paths:

- Local: `<project>/.claude/skills/criar-frontend/`
- Global: `%USERPROFILE%\.claude\skills\criar-frontend\` (Windows) or `~/.claude/skills/criar-frontend/` (Linux/macOS)

Both use `git clone <gitlab-repo> <destino>/criar-frontend`. This only works if the repo contains **only** the skill's files at its root — never the surrounding `.claude/skills/` hierarchy. Do not add wrapper directories.

## Target environment of the generated project

The skill runs commands on the user's machine to scaffold the project. Assume **Windows + PowerShell 5.1** as the primary target (that's the author's environment). The skill's commands should:

- Use the non-interactive form of `npm create vite@latest <name> -- --template react-ts` — never the interactive variant, which would hang.
- Avoid bash-only constructs (`&&`, `$VAR`, `/dev/null`). Use `;` or `if ($?) { ... }` for chaining.

## Component template quirk

The component/page template is intentionally:

```tsx
export const {{Nome}}: React.FC = () => {
  return ()
}
```

The empty `return ()` is **literal** per the user's spec — do not "fix" it to `return null` or `return <></>`. The user fills the JSX in afterward.
