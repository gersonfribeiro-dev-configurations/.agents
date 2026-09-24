# GitHub CLI e GraphQL — Issue Fields e Project V2

## Descoberta obrigatória

- Identificar `owner`, repositório, issue, Project e identidade autorizada antes de executar comandos; nunca fixar organização, número de Project nem IDs de campos/opções.
- Verificar as operações suportadas pelo `gh` instalado (`gh project item-edit --help`, `gh project field-list --help`) e, se necessário, consultar o schema GraphQL atual antes de montar mutations. Para editar as opções do próprio campo use `updateProjectV2Field`; para o valor de um item use `updateProjectV2ItemFieldValue`.
- `gh project field-list <project-number> --owner <org> --format json` fornece IDs dos campos e opções. Confirmar nome, tipo e ID em **cada** Project antes de editar.

## Issue Fields (metadados da issue, pertencentes à organização)

Não confundir os Issue Fields com campos de Project. Descobrir os campos disponibilizados pela organização/repositório (`github_list_issue_fields` ou GraphQL) e usar a ferramenta de issues quando ela cobrir o tipo. Para casos sem cobertura, introspectar os tipos de input de `setIssueFieldValue` na API real e fornecer `issueId`, `fieldId` e valor/opção dessa organização; não usar ID de opção de Project. Um campo `MAJOR` de issue type Release não substitui `Estimate` ou `Status` do Project.

## Project V2 Fields (valores de itens do Project)

```bash
gh project field-list <numero-do-project> --owner <org> --format json
gh project item-list <numero-do-project> --owner <org> --format json
gh project item-edit --help
```

`gh project item-edit` exige **ID do item** e **ID do Project**, além do ID do campo e de uma opção ou valor de tipo compatível; não confundir a URL da issue com o ID do item. Quando a operação desejada não for coberta pela CLI/MCP, consultar IDs reais e usar GraphQL:

```graphql
mutation($projectId: ID!, $itemId: ID!, $fieldId: ID!, $optionId: String!) {
  updateProjectV2ItemFieldValue(input: {
    projectId: $projectId, itemId: $itemId, fieldId: $fieldId,
    value: {singleSelectOptionId: $optionId}
  }) { projectV2Item { id } }
}
```

Para campos numéricos ou de data, confirmar o tipo real antes de enviar `value: {number: ...}` ou `value: {date: "YYYY-MM-DD"}`. Para opções de `Status`, `Size`, `Priority` ou `Effort`, preservar IDs e valores existentes ao alterar sua ordem ou descrição; nunca reaproveitar os IDs do [Template](https://github.com/orgs/aplicacoesBoilerplate/projects/11) em outro Project. Verificar a resposta e consultar novamente o campo/item alterado.
