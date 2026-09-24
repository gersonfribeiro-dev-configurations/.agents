---
name: github-permissions
description: Use when checking PAT scopes, GitHub CLI auth, MCP permissions, or determining which commands must be run by the real user for Projects, Issues, PRs and Packages across the configured GitHub organizations.
---

# Permissões GitHub por operação

## Identidade e escopo

- Confirmar `gh auth status` antes de modificar recursos; a conta ativa pode não ter o mesmo acesso de outra identidade autenticada. Usar credencial autorizada para a organização alvo, com menor privilégio; não trocar a conta ativa do usuário sem necessidade.
- **GitHub App**: permissões dependem da instalação e de cada repositório/organização; verificar Projects, Issues, Pull requests e Contents separadamente.
- **GITHUB_TOKEN em Actions**: permissões por workflow/job (`contents: read` na prévia, `contents: write` somente na publicação pós-merge); acesso a Projects de org pode exigir App/PAT apropriado. Em PRs de forks, não conceder segredos nem execução de código não confiável com permissão de escrita.
- **PAT classic**: `repo` para repositórios privados e, quando necessário, `project`/`read:project` para Projects V2; `write:org` apenas para operações de organização que realmente o exigirem. **PAT fine-grained**: selecionar organizações/repositórios e operações de Projects, Issues, Pull requests e Contents com leitura/escrita compatível com o ato pretendido.
- GitHub CLI, MCP e GraphQL não compartilham necessariamente credenciais. Confirmar permissões do token usado em cada ferramenta, em vez de supor que a API é a causa de um `403`.

## Matriz prática

| Operação | Permissão mínima a confirmar | Ferramenta |
| --- | --- | --- |
| Ler Projects, fields/views e itens | Projects: read | `gh project`, MCP Projects ou `gh api graphql` |
| Editar Status, opções e demais fields do Project | Projects: write | MCP/`gh project` quando suportados, ou `gh api graphql` |
| Criar/editar issue, milestones e Issue Fields | Issues: write no repositório / organização onde o campo vive | MCP Issues, `gh issue` ou API apropriada |
| Criar PR, vincular e solicitar review | Pull requests: write | MCP PR, `gh pr` ou API |
| Publicar tag, GitHub Release ou push | Contents: write e regras de proteção | Git/CLI, workflow pós-merge |
| Publicar/consumir pacote privado | `write:packages`/`read:packages` conforme tipo de token | npm/Maven/CI |

Não atribuir exclusividade ao GraphQL: inspecionar recursos atuais da CLI e MCP. Para options de single-select, conservar IDs e valores do Project. Não confundir Issue Fields da org com Project Fields.

## Operações que exigem o usuário

Concessão/renovação interativa, login e autorização SSO cabem ao usuário real. Se o token autenticado não tiver acesso, reportar qual operação e permissão faltam; não adivinhar nem contornar scopes. Exemplos condicionais para PAT classic:

```bash
gh auth status
gh auth refresh -s project,read:project -h github.com
```

Não imprimir `gh auth token` no terminal nem o conteúdo de `.npmrc`/`settings.xml` (podem expor segredos). Para operações automatizadas usar secret do ambiente e não persistir PAT literal em arquivos versionados. SSH é transporte Git, não substitui permissões de API.
