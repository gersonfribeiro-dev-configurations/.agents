---
name: github-planning
description: Use when planning, creating, updating, reopening, delivering, or reviewing work tracked with GitHub Issues, Milestones, Projects, branches, commits, and pull requests across the configured GitHub organizations.
---

# Planejamento e entrega com GitHub

## Fonte de verdade e ferramentas

- Antes de codificar uma issue, consultar o Project vinculado, milestone, épica, bloqueios, status e repositório. Registrar trabalho novo no Project adequado, sem presumir um único owner para todos os projetos.
- Usar MCP ou GitHub CLI para Issues/PRs e operações de Project efetivamente suportadas; `gh api graphql` para operações sem cobertura. Consultar IDs/opções por Project; nunca reutilizar IDs de outra org. Git/SSH transporta commits, não metadados.
- Distinguir Issue Fields (organização) de Project V2 Fields (itens do Project). Preencher `type`, labels, assignee e milestone na issue e, separadamente, Status, Estimate, Size, Priority, Effort e datas **se existirem** no Project.

## Milestone, épica e sub-issues

- Para uma release, criar/verificar milestone `v<MAJOR>.<MINOR>.<PATCH>` e uma épica de mesmo nome. Épica agrega entregas, não código: type `Release`, label `release`, milestone, `Estimate: 10`, `Size: XL`, `Effort: Team` e datas, onde suportados; Issue Field `MAJOR` somente se existir e corresponder ao caso.
- A sprint é a milestone; sub-issues têm a mesma milestone, título orientado pelo tipo, contexto de entrega, type, labels, assignee e bloqueios `blocked-by`/`blocking` quando aplicáveis. Release não épica não herda automaticamente Estimate 10.
- Descobrir os campos e tipos disponíveis na organização antes de preenchê-los; formulário de issue não atribui automaticamente parent, milestone, Issue Fields ou Project Fields.

### Estimate mede valor agregado, não esforço

| Estimate | Size | Critério |
| ---: | --- | --- |
| 0 | conforme item | homologação |
| 1–5 | XS, S, M, L, XL respectivamente | features por valor agregado |
| 6 | variável | hotfix |
| 7 | conforme escopo | PATCH |
| 8 | conforme escopo | MINOR |
| 9 | conforme escopo | MAJOR |
| 10 | XL | somente épica de Release |

`Effort` é outro campo: não inferir esforço a partir do Estimate; descobrir opções reais de Priority e Effort no Project.

## Branch, worktree e PR

- Criar `release/<milestone>` a partir de `develop` quando necessário. Sub-issue da sprint parte da branch da release em worktree separado; `hotfix/` parte de destino publicado conforme o caso. Confirmar remoto e branch da issue antes do push.
- Uma branch/PR de sub-issue entrega uma issue. PRs de integração `release → develop` e `develop → master` agregam a sprint e não são confundidos com PRs de sub-issue; homologar antes de cada integração.
- PR de sub-issue leva metadados equivalentes aos da issue quando suportados e vínculo em `Development`. Usar palavra-chave de fechamento somente quando o merge na branch padrão de fato deve fechar a issue; senão vincular sem fechá-la prematuramente. Solicitar review do responsável do repositório e seguir branch protection, sem autoaprovação.
- PRs empilhados são permitidos quando há dependência explícita e bases encadeadas; registrar os bloqueios. Sem alteração versionável, não abrir PR artificial; explicar na issue antes de encerrar, quando autorizado.

## Status oficial do Project

Tomar como referência o [Project Template](https://github.com/orgs/aplicacoesBoilerplate/projects/11); confirmar opções e workflows do Project concreto. Ordem: `Backlog`, `Prevented`, `On hold`, `In progress`, `In review`, `Request changes`, `Reopened`, `Ready`, `Done`, `Cancelled`. Não exigir que toda issue passe por todos os estados.

- `Prevented`: item necessário agora, porém bloqueado para começar. Documentar impedimento e a issue bloqueante quando houver.
- `On hold`: fila das próximas iterações, geralmente item prioritário ou que desbloqueia um `Prevented`; não significa bloqueio próprio.
- `In progress`: implementação iniciada; `In review`: PR aguardando análise; `Request changes`: correções solicitadas, sem substituir a review formal.
- `Ready`: PR **aceito/aprovado**, reservado para ser coletado na entrega da release, **antes do merge efetivo**. Não significa pronto para começar.
- `Done`: merge/fechamento concluído de acordo com o fluxo; `Cancelled`: cancelamento/PR encerrado sem merge.

Branch criada, PR aberto, aprovação e merge **podem** acionar workflows de transição; consultar automações reais antes de tratá-las como garantia. Se não ocorrerem, conferir presença no Project e vínculo issue/PR; corrigir após diagnosticar. Evitar alteração manual quando já houver workflow confiável.

## Views e replicação

- Consultar campos e views antes de criar Board, Table ou Roadmap; datas de roadmap dependem de campos presentes. Evitar copiar view ou automação entre Projects sem confirmar suporte da API.
- Para alinhar Projects a um template, comparar campos, opções, ordem, descrições e cores por nome/tipo; preservar IDs de opções existentes e valores dos itens (especialmente ao renomear). Acrescentar o que falta e verificar novamente. Não excluir campos específicos do consumidor sem instrução explícita.
- Metadados do backlog/sprint vivem no GitHub; planejamento técnico local pode morar em `docs/planejamentos-local/`, sem duplicar status e estimativas.

Permissões e exemplos GraphQL: ver `github-permissions` e `github-cli-graphql`.
