---
name: entregas
description: Use when running delivery tasks such as running tests, linting, type-check, build, or setting up ESLint, Prettier, commitlint, Husky, and validating code quality.
---

# Skill: Entregas - Fluxo Sprint/Milestone

## 1 Testes automatizados

- **Backend:** Kafka para testes.
- **Frontend:** Playwright.
- Novos recursos, refatorações ou integrações exigem testes.

## 2 Validacao de código

- SonarQube MCP, ESLint frontend.

## 3 Validacoes finais

- Type-check, docker-compose, consoles sem erros, testes no compose.

## 4 Build

Toda entrega com build (frontend e/ou backend).

## 5 Git Flow e Pull Requests - Regras imutáveis

- Uma branch resolve **uma única issue**, um PR entrega **uma única issue**.
- Árvore permitida: `master`, `develop`, `feature/<nome>`, `hotfix/<nome>`, `release/<versao>`. Nunca `fix/`.
- `feature/` para `Feature/Bug/Task`; `hotfix/` exclusivo para `Hotfix`; `release/` para versão.
- Branch da issue parte de `release/v0.0.1` quando a issue pertence a essa Milestone. Fora de release, parte de `develop`. Nunca de `master`.
- Isolar em **worktree** (ver `git-worktree`). Publicar branch no remoto organizacional antes do PR.
- `git remote -v` deve apontar para `aplicacoesBoilerplate/<repo>`.
- Abrir PR para a branch de origem (`feature/*` -> `release/v0.0.1`, `release/*` -> `develop`, `develop` -> `master`).
- Relatorio `generate-report` e descricao do PR em Markdown. Titulo humano sem `feat:`.
- Preencher no PR: `assignee`, `labels`, `milestone`, `Project`, `type` (quando suportado), `Estimate/Size` se o Project aceitar para PRs. Confirmar via REST.
- `Development`: em PR para `master` usar `Closed #N`; em PR para `release/*`/`develop` vincular manualmente (`Development > Link issue` ou `addLinkedPullRequestToIssue`).
- Em PRs para `develop`/`release/*` fechar a issue manualmente apos merge.
- Status via workflows do Project: `In Progress` (branch), `In Review` (PR), `Ready` (APPROVED owner), `Done` (merge). Nao substituir por mudanca manual exceto sem permissao e com registro.
- Em repo com unico contribuidor, nao autoaprovar. Registrar evidencia, manter `In Review` ate aprovacao do owner.
- PRs enfileirados quando `blocked-by`: abrir 2+ PRs encadeados e declarar dependencia.
- Sem diff versionavel, nao abrir PR. Justificar em comentario e encerrar issue manualmente se autorizado.

## 6 Garantias

100% implementado e funcional antes de abrir PR, com testes/build passando.

---

# Setup de Ferramentas

## ESLint + Prettier

```bash
npm install -D eslint eslint-plugin-vue prettier eslint-config-prettier \
  eslint-plugin-prettier @eslint/js globals typescript typescript-eslint
```

Scripts `package.json`: `lint`, `lint:fix`, `format`.

## Commitlint + Husky

```bash
npm install -D @commitlint/cli @commitlint/config-conventional husky
npx husky init
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

## Arquivos de configuracao

https://github.com/gersonfribeiro/dev-configurations/tree/main/settings
