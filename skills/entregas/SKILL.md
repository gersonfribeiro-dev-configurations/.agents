---
name: entregas
description: Use when running delivery tasks such as running tests, linting, type-check, build, or setting up ESLint, Prettier, commitlint, Husky, and validating code quality.
---

# Skill: Entregas

## 7.1 Testes automatizados

Implementações de recursos, refatorações ou integrações devem conter testes automatizados:

- **Backend:** Kafka para testes.
- **Frontend:** Playwright.
- Novos recursos: criar casos de teste.
- Refatorações: criar casos de teste para validar a mudança.

## 7.2 Validação de código

- Usar MCP do SonarQube para validação de qualidade e segurança.
- Executar ESLint para o frontend.

## 7.3 Validações finais

- Type-check.
- Validação da aplicação em docker-compose.yml.
- Consoles sem erros.
- Testes automatizados executando com sucesso no compose.

## 7.4 Build

Toda entrega deve ser acompanhada de build de código (frontend e/ou backend).

## 7.5 Git Flow e Pull Requests

- Uma branch resolve uma unica issue e um Pull Request entrega uma unica issue, independentemente do tamanho do escopo.
- A arvore Git permitida e composta por `master`, `develop`, `feature/<nome>`, `hotfix/<nome>` e `release/<versao>`. Nao criar prefixes ou tipos alternativos, como `fix/`.
- Usar `feature/` para issues com type `Feature`, `Bug` ou `Task`; `hotfix/` e exclusivo para issues com type `Hotfix`; usar `release/` para preparar uma versao.
- Criar a branch a partir da branch de origem adequada: `develop` para trabalho regular ou a branch de `release` correspondente quando a issue pertencer a uma release em andamento. Nunca abrir branch de issue a partir de `master`.
- Publicar a branch no remoto antes de abrir o Pull Request.
- Conferir `git remote -v` antes do push. Quando `origin` apontar para fork ou repositório movido, publicar explicitamente a branch no repositório organizacional que receberá o Pull Request e confirmar sua existência nele.
- Abrir o Pull Request para a branch de origem. Nunca entregar uma issue diretamente em `master`.
- Entregar branches de `release` em `develop` por Pull Request e entregar `develop` em `master` tambem por Pull Request.
- Gerar o relatorio pela skill `generate-report` para toda entrega. A descricao do Pull Request deve conter o relatorio completo em Markdown e `Fixed #123` ou `Closed #123`, substituindo `123` pela issue entregue.
- O titulo do Pull Request deve resumir a realizacao para leitura humana. Nao usar prefixos tecnicos como `feat:` ou titulos de merge como `merge: release v1.0.0 in develop`.
- Antes de enviar o Pull Request para revisao, preencher e conferir: assignee, labels, type, fields `Estimate`, `Size`, `Priority` e `Effort`, Project e milestone. Replicar os valores da issue entregue quando aplicavel.
- Confirmar a secao Development: em Pull Requests para a branch padrao, `Fixed #123` ou `Closed #123` cria o vinculo; em Pull Requests para `release/*` ou `develop`, vincular manualmente a unica issue pelo controle **Development > Link issue**.
- Fields e Project de um Pull Request podem exigir as mutations GraphQL `addProjectV2ItemById` e `updateProjectV2ItemFieldValue`; quando a integracao nao as expuser, solicitar a permissao e a capacidade antes da revisao, sem registrar esses metadados apenas no texto do PR.
- O GitHub fecha uma issue por `Fixed #123` ou `Closed #123` somente quando o Pull Request e mesclado na branch padrao. Para entregas em `release/*` ou `develop`, fechar manualmente a issue como concluida depois do merge e da validacao, com comentario que registre a limitacao.
- Usar os workflows configurados no proprio Project para as transicoes de Status. A issue e o Pull Request devem estar adicionados ao mesmo Project para que seus eventos de revisao, merge ou fechamento possam acionar as regras previstas; nao criar GitHub Actions quando o Project ja cobrir esse fluxo.
- Manter o item em `In review` enquanto o Pull Request estiver aberto e sem aprovacao. O workflow de aprovacao deve movê-lo para `Ready`; `Done` e reservado para merge ou fechamento.
- Em repositorio com um unico contribuidor, nao exigir autoaprovacao. Depois de validar testes, lint, type-check, build e o relatorio, registrar a evidencia em comentario e mover manualmente o item do Project de `In review` para `Ready` antes do merge.
- Code review e aprovacao sao manuais. Nao aprovar, fazer merge ou ignorar requisitos de revisao sem solicitacao explicita.
- Excecoes sem alteracao versionavel, como atualizacoes de configuracao remota ou skills ja publicadas fora do repositorio, devem ser justificadas em comentario na issue antes do encerramento manual.
- Registrar no Pull Request as validações aprovadas e bloqueios preexistentes que impeçam validações globais, sem incluir arquivos fora do escopo para contorná-los.

## 7.6 Garantias

Garantir que todas as tarefas foram executadas, estão funcionais e em conformidade com as especificações.

---

# Setup de Ferramentas

## ESLint + Prettier

```bash
npm install -D eslint eslint-plugin-vue prettier eslint-config-prettier \
  eslint-plugin-prettier @eslint/js globals typescript typescript-eslint
```

Scripts no `package.json`:

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier . --write"
  }
}
```

## Commitlint + Husky

```bash
npm install -D @commitlint/cli @commitlint/config-conventional husky
npx husky init
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

Script no `package.json`:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

## Arquivos de configuração padronizados

Disponíveis em: https://github.com/gersonfribeiro/dev-configurations/tree/main/settings

- ESLint (`.eslintrc.cjs`)
- Prettier (`prettier.config.cjs`)
- commitlint (`commitlint.config.cjs`)
- EditorConfig (`.editorconfig`)
- VSCode Settings
