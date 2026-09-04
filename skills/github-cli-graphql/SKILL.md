# Exemplos de comandos para criar issues usando o GitHub CLI

## Crie a issue e salve a URL resultante em uma variável

ISSUE_URL=$(gh issue create --repo aplicacoesBoilerplate/boilerplate-cli --title "Definir contrato de manifesto DX dos packages" --label enhancement --assignee agentegersonfribeiro-AI --project "CLI Boilerplate Go - DX" --milestone v0.0.1)

### Atualizando os Project Fields

gh project item-edit 10 --owner aplicacoesBoilerplate --url $ISSUE_URL --field "Priority" --value "High"

1. 10 É o código do projeto, não da issue;
2. Os fields mudam a tag --value com variações como number, text...

## Usando gh api graphql para issue fields

### Descobrindo os ID's no banco do GitHub

Diferente das APIs tradicionais, o GraphQL não usa o número #6 ou a string "Medium". Você precisa consultar os IDs únicos (hash) da issue, do campo na organização e da opção desejada. Execute a query abaixo:

```bash
gh api graphql -F owner="aplicacoesBoilerplate" -F repo="boilerplate-cli" -F issueNumber=6 -f query='
  query($owner: String!, $repo: String!, $issueNumber: Int!) {
    repository(owner: $owner, name: $repo) {
      issue(number: $issueNumber) { id }
    }
    organization(login: $owner) {
      issueFields(first: 10) {
        nodes {
          ... on IssueFieldSingleSelect {
            id
            name
            options { id name }
          }
        }
      }
    }
  }'
```

### Mutação da issue via API

O objeto value recebe atributos específicos dependendo do tipo do campo (neste caso, singleSelectOptionId para um menu de seleção):

```bash
gh api graphql -F issueId="SEU_ISSUE_ID" -F fieldId="SEU_FIELD_ID" -F optionId="SEU_OPTION_ID" -f query='
  mutation($issueId: ID!, $fieldId: ID!, $optionId: String) {
    setIssueFieldValue(input: {
      issueId: $issueId,
      fieldId: $fieldId,
      value: { singleSelectOptionId: $optionId }
    }) {
      clientMutationId
    }
  }'
```

#### Tipos de Injeção no Objeto value:

* **Single Select** (Priority/Effort): { singleSelectOptionId: $optionId }.
* **Text**: { text: "Seu texto" }.
* **Number** (Estimate): { number: 5 }.
* **Date** (Start/Target date): { date: "2026-09-04" }.