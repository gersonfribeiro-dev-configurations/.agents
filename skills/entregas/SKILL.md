---
name: entregas
description: Use when running delivery tasks such as running tests, linting, type-check, build, or setting up ESLint, Prettier, commitlint, Husky, and validating code quality.
---

# Entregas por stack e release

## Verificações pertinentes

- Identificar a stack e os comandos configurados antes de executar lint/formatação, testes, análise estática, type-check e build. Rodar os checks necessários para a mudança, sem exigir build em entrega documental nem Kafka/Playwright em projeto que não os usa.
- Backend: testar regras, integração e migração quando alteradas; Kafka somente para fluxos de mensagens. Frontend: testes de componentes e Playwright quando houver comportamento E2E/UI pertinente. Workflows/templates: validar YAML, contratos e cenários em repositório de ensaio.
- Quando SonarQube estiver configurado, disparar análise por PR e verificar quality gate de código novo. Gate reprovado ou inconclusivo não equivale a aprovado; exigir check no ruleset/branch protection quando aplicável. `Request changes` no Project é sinalização visual, não substitui review nem protege merge por si só.
- Informar verificações realmente executadas e bloqueios reais; não afirmar que todos os projetos têm a mesma infraestrutura.

## Git, release e PR

- Para sub-issue de milestone, partir de `release/<milestone>` derivada de `develop`, usar branch e worktree isolados; hotfix segue o destino publicado. Confirmar remoto real, sem owner fixo. Uma branch/PR de sub-issue entrega uma issue; PR de integração `release → develop` e `develop → master` agrega a sprint, com homologação antes de cada merge.
- Vincular PR à issue e preencher metadados equivalentes quando suportados; pedir review ao responsável configurado. Não autoaprovar. Documentar realização, fontes, teste e novidade com `generate-report`.
- Conferir os Status existentes e workflows do Project: `In progress` após início, `In review` durante review, `Ready` após aceitação do PR **e antes do merge** (aguarda coleta na release), `Done` somente após integração/fechamento. `Prevented` é trabalho necessário impedido de começar; `On hold` é fila de próximas iterações, inclusive desbloqueantes/prioritários. Não forçar transição manual se a automação real já cobre o evento.
- Sem diff versionável, não abrir PR artificial. Comentar o motivo e encerrar issue quando autorizado.

## Ferramentas locais

- ESLint/Prettier, commitlint e Husky só devem ser configurados quando aplicáveis à stack e compatíveis com o repositório; revisar versões e docs oficiais antes de instalar. Hooks não substituem checks obrigatórios na CI.
- Para CI/CD e GitHub Projects, consultar também `cicd`, `github-planning` e `github-permissions`.
