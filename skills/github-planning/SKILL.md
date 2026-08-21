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
- Criar issues com autoria do usuario autenticado e atribui-lo quando aplicavel.
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
- Criar a branch de issue a partir de `develop` ou da branch de `release` correspondente. Nunca partir de `master`, nem abrir Pull Request de issue diretamente para `master`.
- Publicar a branch no remoto e abrir o Pull Request para a branch de origem. Branches de `release` devem ser entregues em `develop` por Pull Request; `develop` deve ser entregue em `master` por Pull Request.
- Verificar se `origin` representa o repositório organizacional. Se apontar para fork ou repositório movido, enviar a branch também ao repositório que receberá o Pull Request e confirmar o SHA remoto antes da abertura.
- Gerar o relatorio completo com a skill `generate-report` e usá-lo como descricao em Markdown do Pull Request. Incluir `Fixed #123` ou `Closed #123`, substituindo `123` pelo numero da issue entregue, para permitir o fechamento automatico.
- O titulo do Pull Request deve resumir a realizacao, sem prefixos como `feat:` e sem titulos tecnicos de merge.
- Review e aprovacao sao exclusivamente manuais. Nunca aprovar ou fazer merge em nome do usuario sem solicitacao explicita.
- Ao reabrir uma issue, adicionar comentario com a justificativa. O workflow do Project move o status; a entrega posterior deve incluir o relatorio habitual.
- Sem alteracao versionavel no repositorio, nao criar Pull Request artificial. Registrar a justificativa em comentario e encerrar manualmente a issue quando o usuario autorizar.
- O `Status` do Project e independente do estado da issue: mover `Backlog` para `In progress` ao iniciar a branch, `In progress` para `In review` depois das validações e antes do Pull Request, e aguardar os workflows para `Ready` e o merge para `Done`.
- Quando a integração não expuser escrita no `Status`, registrar a transição pretendida em comentário, informar a limitação e nunca declarar que o campo foi efetivamente alterado.

## Permissoes do PAT

Para Projects V2 de organizacao com fine-grained PAT, usar `Organization permissions > Projects: Read-only` para consultas e `Read and write` quando for alterar Projects. Para criar e atualizar issues, conceder `Repository permissions > Issues: Read and write` aos repositorios selecionados. O acesso ao Project tambem precisa estar liberado para o usuario ou equipe.
