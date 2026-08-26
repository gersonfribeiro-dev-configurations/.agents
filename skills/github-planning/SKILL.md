---
name: github-planning
description: Use when planning, creating, updating, reopening, delivering, or reviewing work tracked with GitHub Issues, Milestones, Projects, branches, commits, and pull requests in the aplicacoesBoilerplate organization.
---

# Skill: Governanca GitHub MCP

## Planejamento obrigatorio

Antes de propor ou iniciar uma feature no modo plano, consultar o GitHub pelo MCP e identificar:

- O Project, incluindo `Status`, `Estimate` e `Size` quando disponiveis.
- A milestone aberta aplicavel.
- A issue pai da release e sua arvore de sub-issues.
- Types, labels, fields, dependencias, issues abertas e milestones abertas relacionadas.
- A arvore Git, tags existentes, servicos em andamento e trabalho futuro pendente.

Se a feature nao tiver milestone e issue pai adequadas, criar primeiro a estrutura de release. A issue pai deve ser uma `Release` MAJOR, possuir label `release`, milestone e estimativa `10`.

Criar milestones somente depois de avaliar a arvore Git, tags, entregas pendentes e o planejamento das demais milestones abertas. Nao criar milestones por conveniencia isolada de uma issue.

## Modelo de issues

- A issue pai e vinculada a uma milestone; as sub-issues devem permanecer na mesma milestone e na arvore correta.
- Criar issues, comentarios e Pull Requests exclusivamente com a autoria da conta autenticada pelo PAT; nunca personificar outro usuario. Atribuir a conta autenticada quando aplicavel.
- Preencher type, labels, fields, titulo e descricao enriquecida. A descricao pode conter trechos do plano; comentarios adicionais sao permitidos quando a divisao melhora a rastreabilidade.
- Usar `Release` para mudancas semanticas de versao, `Feature` para funcionalidade, `Bug` para defeito, `Hotfix` para correcao emergencial e `Task` apenas para trabalho tecnico sem melhor type de dominio.

## Estimate e Size

`Estimate` e `Size` representam a agregacao de esforco do escopo rastreado; nao sao metricas de tamanho do repositorio nem substituem `Priority` ou `Effort`.

| Estimate | Size | Uso |
| ---: | --- | --- |
| 1 | XS | Feature de escopo muito pequeno |
| 2 | S | Feature pequena |
| 3 | M | Feature media |
| 4 | L | Feature grande |
| 5 | XL | Feature muito grande |
| 6 | XL | Hotfix, exclusivamente |
| 7 | XL | Feature associada a release PATCH |
| 8 | XL | Feature associada a release MINOR |
| 9 | XL | Feature associada a release MAJOR |
| 10 | XL | Issue pai de release MAJOR e milestone |

Preencher ambos os fields em toda issue. Valores de `6` a `10` representam agregacoes de escopo; o `Size` permanece `XL` por ser a maior classificacao categorica disponivel.

## Execucao e entrega

- Executar somente uma issue por iteracao.
- Aplicar a skill `entregas` ao iniciar a entrega.
- Uma branch resolve uma unica issue e um Pull Request entrega uma unica issue, independentemente do tamanho do escopo.
- Usar exclusivamente `master`, `develop`, `feature/<nome>`, `hotfix/<nome>` e `release/<versao>`. Nunca criar prefixes alternativos, como `fix/`.
- Usar `feature/` para issues com type `Feature`, `Bug` ou `Task`; `hotfix/` e exclusivo para issues com type `Hotfix`; usar `release/` para preparacao de versao.
- Quando a issue pertencer a uma release que ja tenha branch `release/<versao>`, criar a branch da issue obrigatoriamente a partir dessa branch e abrir o Pull Request de volta para ela. Usar `develop` somente para itens fora de uma release ativa. Nunca partir de `master`, nem abrir Pull Request de issue diretamente para `master`.
- Publicar a branch no remoto e abrir o Pull Request para a sua branch de origem. Branches de `release` devem ser entregues em `develop` por Pull Request; `develop` deve ser entregue em `master` por Pull Request.
- Verificar se `origin` representa o repositório organizacional. Se apontar para fork ou repositório movido, enviar a branch também ao repositório que receberá o Pull Request e confirmar o SHA remoto antes da abertura.
- Gerar o relatorio completo com a skill `generate-report` e usá-lo como descricao em Markdown do Pull Request. Usar `Closed #123` somente quando a PR aponta para a branch padrão: o GitHub ignora palavras-chave de fechamento em PRs para `release/*` ou `develop`.
- O titulo do Pull Request deve resumir a realizacao, sem prefixos como `feat:` e sem titulos tecnicos de merge.
- Antes de solicitar revisao, o Pull Request deve ter a conta autenticada pelo PAT como assignee, labels e milestone, além do Project. Os valores devem corresponder aos da issue entregue quando aplicavel. `Estimate`, `Size`, `Priority` e `Effort` pertencem à issue; não replicá-los na PR se o Project não aceitar esses campos para pull requests. Usar REST para assignee, labels e milestone e conferir esses metadados pela API depois da criação; não considerar o Pull Request pronto enquanto algum campo obrigatório estiver ausente.
- Todo Pull Request deve solicitar revisao do usuario proprietario do workspace. A aprovacao desse usuario, registrada como `APPROVED`, e obrigatoria antes de merge; o agente nunca pode se autoaprovar, remover esse requisito ou fazer merge sem solicitacao explicita do usuario.
- Confirmar em Development uma unica issue vinculada. Para PRs na branch padrão, `Fixed #123` ou `Closed #123` cria o vínculo. Para PRs em `release/*` ou `develop`, vincular manualmente a única issue por **Development > Link issue**, ou pela mutation GraphQL `addLinkedPullRequestToIssue` quando essa capacidade estiver exposta. A API REST não oferece essa operação.
- Para incluir um Pull Request no Project e preencher seus fields, usar `addProjectV2ItemById` e `updateProjectV2ItemFieldValue`. Se essas mutations nao estiverem disponiveis na integracao, informar o bloqueio e solicitar a capacidade antes da revisao.
- Para PRs em branch não padrão, não usar `Closed #N` como mecanismo de vínculo: registrar a issue no relatório sem palavra-chave de fechamento e criar o vínculo em Development manualmente ou por GraphQL.
- Ao criar a branch, adicionar a issue ao Project e definir o Status como `In progress`. Depois de vincular a PR em Development, validar se a abertura acionou o workflow para `In review`; uma aprovacao `APPROVED` deve mover para `Ready`; o merge deve mover para `Done`. Antes de iniciar outra issue, validar esses tres gatilhos no Project e corrigir os workflows se necessario. Nao substituir os workflows por mudancas manuais de Status, exceto quando a integracao nao tiver permissao de escrita e a limitacao for registrada.
- Apos cada merge, verificar se os workflows do Project efetivaram a transicao do item para `Done` e o encerramento da issue. Se a transicao nao ocorrer, diagnosticar a causa (item ausente do Project, vinculo por palavra errada como `Refs`, workflow desabilitado) antes de qualquer ajuste manual, e registrar a correcao aplicavel na skill ou nos workflows para evitar recorrencia.
- O item deve ficar em `In review` enquanto o PR estiver aberto e sem aprovacao. Aprovacao real de review, com estado `APPROVED` e nao apenas `COMMENTED`, aciona o workflow para `Ready`; `Done` somente depois de merge ou fechamento.
- Em repositorio com unico contribuidor, o autor nao deve autoaprovar. Concluidas as validacoes, registrar a evidencia no PR e manter o item em `In review` ate a aprovacao do usuario; o workflow de `APPROVED` deve movê-lo para `Ready`, e o merge ou fechamento mantem a transicao para `Done` pelos workflows do Project.
- Review e aprovacao sao exclusivamente manuais e pertencem ao usuario solicitante. Nunca aprovar, dispensar a revisao ou fazer merge em nome do usuario sem solicitacao explicita.
- Ao reabrir uma issue, adicionar comentario com a justificativa. O workflow do Project move o status; a entrega posterior deve incluir o relatorio habitual.
- Sem alteracao versionavel no repositorio, nao criar Pull Request artificial. Registrar a justificativa em comentario e encerrar manualmente a issue quando o usuario autorizar.
- O `Status` do Project e independente do estado da issue: a transicao para `In progress` e feita ao iniciar a branch; as transicoes seguintes pertencem aos workflows de abertura do Pull Request, aprovacao e merge.
- Quando a integração não expuser escrita no `Status`, registrar a transição pretendida em comentário, informar a limitação e nunca declarar que o campo foi efetivamente alterado.
- Depois que um Pull Request for aberto, enviar commits adicionais para a mesma branch quando forem correcoes do mesmo escopo; o Pull Request sera atualizado automaticamente. Nunca fecha-lo e recria-lo apenas para incluir novas alteracoes. Fechar e substituir somente se a base, o historico ou o escopo estiverem incorretos e nao puderem ser corrigidos sem reescrita proibida da branch.

## Permissoes do PAT

Para Projects V2 de organizacao com fine-grained PAT, usar `Organization permissions > Projects: Read-only` para consultas e `Read and write` para adicionar itens ou alterar fields e Status. Para criar e atualizar issues e os metadados nativos de Pull Requests, conceder `Repository permissions > Issues: Read and write` e `Repository permissions > Pull requests: Read and write` aos repositorios selecionados. A segunda permissao e necessaria para solicitar reviewers. O acesso ao Project tambem precisa estar liberado para o usuario ou equipe.
