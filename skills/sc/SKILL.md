---
name: sc
description: Use when creating, evolving, or maintaining an application integrated with SoftwareCenter, including BFF sessions, tenants, contracts, and RBAC.
---

# Integração com a SoftwareCenter (SC)

## Objetivo

Use este skill ao criar, evoluir ou manter uma aplicação integrada à SoftwareCenter (SC). A SC é o control plane de identidade, tenants, contratos, liberações de recursos e RBAC.

Documente e implemente somente os contratos descritos aqui. Não invente endpoints, claims, scopes ou fluxos que não estejam explicitamente disponíveis.

## Decisão arquitetural

Cada aplicação integrada mantém seu próprio frontend, backend/BFF e sessão. A SC não compartilha o cookie da sessão do painel com aplicações clientes.

```text
Frontend da aplicação
  | cookie próprio HttpOnly + proteção CSRF
  v
Backend/BFF da aplicação
  | sessão BFF, OAuth/OIDC ou catálogo, conforme o caso
  v
SoftwareCenter
  |
  v
Identidade, tenant, contrato, liberação e RBAC
```

### Regras obrigatórias

- O navegador nunca recebe senha da SC, credencial técnica, access token, refresh token ou segredo de cliente.
- O BFF identifica a aplicação por sua configuração e credencial técnica, nunca por um `applicationId` enviado pelo navegador.
- A sessão local é da aplicação. Use cookie `HttpOnly`, `Secure` em HTTPS e `SameSite=Lax` ou mais restritivo.
- Não registre senha, token de ativação, OTP, `sessionId` ou segredo técnico em logs, traces, métricas, filas ou respostas de erro.
- Não use `localStorage` ou `sessionStorage` para tokens, credenciais ou sessão autenticada.
- O tenant é parte obrigatória do contexto de acesso. Para aplicações não exclusivas, o BFF informa o `tenantSubdominio`; para aplicações exclusivas, a SC resolve o tenant beneficiário configurado.

## Modelo de acesso

| Conceito | Responsabilidade |
| --- | --- |
| Identidade | Usuário central, com status `PENDING`, `ACTIVE` ou `BLOCKED`. |
| Tenant | Organização identificada pelo `subdominio`. |
| Membership | Vínculo usuário-tenant, com papel `OWNER`, `ADMIN` ou `MEMBER`. |
| Aplicação | Recurso integrado à SC. Cada aplicação possui seus próprios cargos e permissões e pode ser exclusiva de um tenant beneficiário. |
| Permissão | Chave estável da capacidade da aplicação, por exemplo `invoice.facturas.read`. |
| Cargo | Conjunto de permissões de uma aplicação para um tenant. |
| Atribuição | Vínculo entre membership, aplicação e cargo. |
| Capability | Capacidade administrativa do membership: `MEMBERS_MANAGE` e `RBAC_MANAGE`. |
| Contrato e liberação | Determinam se o tenant beneficiário pode acessar o recurso/aplicação. |

### Exclusividade de aplicação

Uma aplicação pode ser cadastrada com `exclusiva = true` e um `idTenantBeneficiarioExclusivo`. Esse tenant é selecionado entre os tenants ativos no cadastro administrativo do projeto.

- O tenant owner continua sendo o fornecedor da aplicação.
- O tenant beneficiário exclusivo deve ser diferente do tenant owner.
- Não é possível habilitar ou trocar a exclusividade enquanto houver liberação ativa para outro tenant.
- Contratos novos e ativações de contratos são rejeitados quando incluírem recurso de aplicação exclusiva para outro tenant.
- Em sessões BFF de aplicação exclusiva, a SC ignora `tenantSubdominio` e usa o tenant beneficiário cadastrado.
- Aplicações não exclusivas mantêm o fluxo multi-tenant existente e exigem `tenantSubdominio`.

Uma sessão só é válida quando a identidade, o membership, o tenant, a aplicação, o recurso, o contrato/liberação, a atribuição, o cargo e suas permissões permanecem ativos e coerentes.

Um `OWNER` ou `ADMIN` recebe as capabilities `MEMBERS_MANAGE` e `RBAC_MANAGE`. Um `MEMBER` não recebe capabilities.

## Integração por BFF

Use este contrato para login com senha ou Google em aplicações web, mobile com backend próprio e APIs que precisem criar uma sessão de usuário. O BFF é o único consumidor dos endpoints de sessão da SC.

### Credencial BFF

Os endpoints de sessão exigem HTTP Basic:

```http
Authorization: Basic base64(bffClientId:bffClientSecret)
```

A credencial BFF pertence a uma única aplicação e precisa estar ativa. Ela é independente de clientes OIDC e de clientes técnicos de catálogo. O segredo só é retornado em texto claro no momento de sua rotação pelo painel da SC; persista-o exclusivamente no cofre de segredos/configuração protegida do backend.

### Criar sessão com senha

`POST /api/v1/integracoes/sessoes`

```json
{
  "email": "usuario@empresa.com",
  "senha": "nao-registrar-em-logs",
  "tenantSubdominio": "empresa"
}
```

Resposta `201 Created`:

```json
{
  "sessionId": "identificador-opaco-de-43-caracteres",
  "expiresAt": "2026-07-31T20:00:00Z",
  "context": {
    "userId": 10,
    "tenantId": 20,
    "tenantSubdomain": "empresa",
    "membershipId": 30,
    "applicationId": 40,
    "roleId": 50,
    "roleName": "Administrador",
    "roleIcon": "mdi-shield-account",
    "permissions": ["invoice.facturas.read"],
    "capabilities": ["RBAC_MANAGE", "MEMBERS_MANAGE"]
  }
}
```

`tenantSubdominio` é obrigatório apenas para aplicações não exclusivas. Para aplicações exclusivas, ele pode ser omitido e qualquer valor eventualmente informado é ignorado em favor do tenant beneficiário configurado.

O usuário precisa estar `ACTIVE`, possuir e-mail verificado e ter senha válida. A SC aplica limite de tentativas por aplicação, e-mail normalizado e IP. Não revele se a falha foi causada por senha, usuário, tenant ou regra de acesso.

### Criar sessão com Google

`POST /api/v1/integracoes/sessoes/google`

```json
{
  "credential": "id-token-publico-do-google",
  "tenantSubdominio": "empresa"
}
```

A resposta é idêntica à sessão com senha. A SC valida assinatura, audience configurada, issuer Google e `email_verified`. Se Google não estiver configurado na SC, a resposta é `503 Service Unavailable`.

### Revalidar e revogar sessão

`GET /api/v1/integracoes/sessoes/{sessionId}` retorna `200 OK`:

```json
{
  "expiresAt": "2026-07-31T20:00:00Z",
  "context": {
    "userId": 10,
    "tenantId": 20,
    "tenantSubdomain": "empresa",
    "membershipId": 30,
    "applicationId": 40,
    "roleId": 50,
    "roleName": "Administrador",
    "roleIcon": "mdi-shield-account",
    "permissions": ["invoice.facturas.read"],
    "capabilities": ["RBAC_MANAGE", "MEMBERS_MANAGE"]
  }
}
```

Use `DELETE /api/v1/integracoes/sessoes/{sessionId}` para revogar a sessão na SC. A operação retorna `204 No Content` e é idempotente.

A revalidação não estende a expiração da sessão na SC. Se a sessão estiver expirada, revogada ou alguma regra de acesso deixar de ser válida, a SC responde `401 Unauthorized` e revoga a sessão. Nesse caso, o BFF deve encerrar também a sessão local.

### Interpretação de falhas

| Status | Tratamento no BFF |
| --- | --- |
| `401 Unauthorized` | Credencial BFF ou sessão inválida; encerre a sessão local e exija login quando aplicável. |
| `403 Forbidden` | Identidade autenticada sem acesso efetivo; não exponha o motivo específico ao usuário. |
| `429 Too Many Requests` | Limite de tentativas de login; respeite `Retry-After` quando presente e aplique feedback genérico. |
| `503 Service Unavailable` | Provedor Google indisponível ou não configurado; ofereça autenticação alternativa se existir. |

### Responsabilidades do BFF

1. Receber credenciais do frontend apenas no endpoint local de login.
2. Encaminhar a requisição à SC pelo canal servidor-servidor com a credencial BFF.
3. Persistir no servidor apenas o `sessionId` opaco e os dados mínimos de contexto necessários.
4. Emitir cookie local HttpOnly e expor um endpoint local, como `GET /auth/session`, que devolva somente estado seguro para a interface.
5. Revalidar periodicamente ou antes de operações sensíveis usando o `sessionId`.
6. Revogar na SC e invalidar o cookie local no logout.

## Identidade e credenciais

Estes endpoints são públicos e pertencem ao domínio de identidade da SC. Uma aplicação pode direcionar usuários a esses fluxos quando sua experiência exigir cadastro ou recuperação centralizada.

### Cadastro e verificação de e-mail

`POST /api/v1/public/cadastro`

```json
{
  "nome": "Nome do usuário",
  "email": "usuario@empresa.com",
  "senha": "senha-com-pelo-menos-10-caracteres"
}
```

Retorna `201 Created` com a identidade em estado `PENDING`. A SC envia o token de verificação por e-mail.

`POST /api/v1/public/verificacoes-email`

```json
{
  "token": "token-recebido-por-email"
}
```

Após a verificação, a identidade passa a `ACTIVE`. Login local por senha exige e-mail verificado.

### Recuperação de senha

Solicite recuperação em `POST /api/v1/public/recuperacoes-senha`:

```json
{
  "email": "usuario@empresa.com"
}
```

A resposta é sempre `202 Accepted` com mensagem genérica para não enumerar identidades. A SC envia um OTP de seis dígitos quando aplicável.

Valide o OTP em `POST /api/v1/public/recuperacoes-senha/verificacoes`:

```json
{
  "email": "usuario@empresa.com",
  "otp": "123456"
}
```

Redefina a senha em `POST /api/v1/public/redefinicoes-senha`:

```json
{
  "email": "usuario@empresa.com",
  "otp": "123456",
  "novaSenha": "nova-senha-com-pelo-menos-10-caracteres"
}
```

O OTP expira em 10 minutos, possui limite de tentativas e, depois de uma redefinição válida, as sessões/autorizações existentes são encerradas.

## Provisionamento e ativação de acesso

Não existe um fluxo público de "solicitação de acesso". O fluxo disponível é o provisionamento emitido por um administrador autorizado do tenant.

O painel ou cliente autorizado emite o provisionamento com `POST /api/v1/tenants/contexto-atual/provisionamentos`:

```json
{
  "email": "novo.usuario@empresa.com",
  "idCargoInicial": 50,
  "idAplicacaoOrigem": 40
}
```

A SC envia um link de ativação de uso único. O token expira em sete dias.

### Consulta e cadastro por ativação

- `GET /api/v1/public/ativacoes-acesso/{token}` consulta dados seguros da ativação, como tenant, subdomínio, e-mail mascarado e expiração.
- `POST /api/v1/public/ativacoes-acesso/{token}/cadastro` recebe `{ "nome": "...", "senha": "..." }`, cria ou completa a identidade e ativa o membership.
- `POST /api/v1/ativacoes-acesso/{token}/aceite` permite que uma identidade SC já autenticada aceite a ativação. O e-mail autenticado deve ser o mesmo do provisionamento.

Uma ativação bem-sucedida cria ou ativa o membership do tenant e atribui o cargo inicial para a aplicação correspondente.

## OAuth2, OIDC e token delegado

A SC é um Authorization Server OIDC. O discovery e endpoints OAuth2 são servidos pelo issuer configurado na SC; as chaves públicas ficam em `/oauth2/jwks`.

### Token de usuário

Clientes OIDC de aplicação podem usar `authorization_code` com PKCE e `refresh_token`, conforme o cliente registrado. Tokens de usuário incluem o contexto de acesso:

```json
{
  "sub": "10",
  "aud": ["audience-da-aplicacao"],
  "tenant_id": 20,
  "tenant_subdomain": "empresa",
  "membro_tenant_id": 30,
  "application_id": 40,
  "role_id": 50,
  "role_name": "Administrador",
  "role_icon": "mdi-shield-account",
  "permissions": ["invoice.facturas.read"],
  "capabilities": ["RBAC_MANAGE", "MEMBERS_MANAGE"],
  "name": "Nome do usuário",
  "email": "usuario@empresa.com",
  "has_avatar": false,
  "jti": "identificador-unico"
}
```

O nome do claim de membership em JWT é `membro_tenant_id`. Não use `membership_id` para tokens OAuth/OIDC.

### Cadastro do cliente OIDC e CORS

Cadastre o cliente OIDC pela API de catálogo da SC. Os valores ficam persistidos no banco da SC; não adicione configurações específicas da aplicação consumidora ao ambiente da SC.

`POST /api/v1/aplicacoes/{idAplicacao}/clientes-oidc`

```json
{
  "clientId": "minha-aplicacao",
  "clientSecret": "segredo-gerado-e-armazenado-no-backend-consumidor",
  "redirectUris": ["https://app.exemplo.com/login/oauth2/code/software-center"],
  "postLogoutRedirectUris": ["https://app.exemplo.com"],
  "origensCors": ["https://app.exemplo.com"]
}
```

Cada origem CORS deve conter somente scheme, host e porta, por exemplo `https://app.exemplo.com`. Não inclua caminho, query, fragmento ou credenciais. A SC permite uma origem de aplicação somente quando ela estiver cadastrada para alguma aplicação ativa; não use uma lista global de origens em variáveis de ambiente.

### Token delegado para RBAC

Para administrar RBAC pela API integrada, o BFF troca um token de usuário por token de audience `sc-management`. O token delegado tem TTL curto e exige capability `RBAC_MANAGE` no token de origem.

O token delegado precisa conter audience `sc-management`, `application_id` igual ao recurso no path e scope apropriado. Os scopes disponíveis nos endpoints atuais são:

| Endpoint | Scope |
| --- | --- |
| `GET /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/membros` | `sc.members.read` |
| `GET /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/permissoes` | `sc.rbac.read` |
| `GET /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/cargos` | `sc.rbac.read` |
| `GET /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/atribuicoes` | `sc.rbac.read` |
| `POST /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/cargos` | `sc.rbac.write` |
| `PUT /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/cargos/{idCargo}` | `sc.rbac.write` |
| `POST /api/v1/integracoes/rbac/aplicacoes/{idAplicacao}/atribuicoes` | `sc.rbac.write` |

Exemplo de cargo:

```json
{
  "nome": "Operador",
  "descricao": "Opera faturas",
  "icone": "mdi-account-cog",
  "idsPermissoes": [101, 102],
  "breakGlass": false,
  "justificativa": "Cargo operacional inicial"
}
```

Exemplo de atribuição:

```json
{
  "idMembroTenant": 30,
  "idCargo": 50
}
```

O ator precisa manter autorização efetiva para administrar RBAC e não pode delegar permissões acima das suas permissões efetivas.

## Catálogo e manifesto da aplicação

O manifesto é a fonte declarativa das permissões e rotas da aplicação. Use-o em automações de deploy/onboarding para manter o catálogo da SC sincronizado.

### Credencial técnica de catálogo

Obtenha um access token por `client_credentials` com a credencial de catálogo da aplicação. O token técnico deve ter:

- audience `sc-catalog`;
- scope `sc.catalog.sync`;
- claim `application_id` da aplicação vinculada;
- claim `client_id` do cliente técnico vinculado.

Use `Authorization: Bearer {accessToken}` nos endpoints abaixo. Esta credencial é diferente da credencial BFF.

### Contrato do manifesto

```json
{
  "schemaVersion": 1,
  "applicationKey": "invoice",
  "version": "1.0.0",
  "permissions": [
    {
      "key": "invoice.facturas.read",
      "name": "Consultar faturas",
      "description": "Permite consultar faturas.",
      "group": "Faturas",
      "icon": "mdi-file-document-outline"
    }
  ],
  "routes": [
    {
      "kind": "FRONTEND",
      "path": "/faturas",
      "name": "Faturas",
      "permission": "invoice.facturas.read"
    },
    {
      "kind": "API",
      "method": "GET",
      "path": "/api/faturas",
      "name": "Consultar faturas",
      "permission": "invoice.facturas.read"
    }
  ]
}
```

Regras do schema:

- `schemaVersion` é obrigatoriamente `1`.
- `applicationKey` precisa ser igual à chave do recurso da aplicação registrada na SC.
- A chave de permissão deve ser única no manifesto.
- Cada rota deve referenciar uma permissão presente no mesmo manifesto.
- Rotas `API` exigem `method`; rotas `FRONTEND` não usam método.
- Rotas são únicas por `kind`, `method` e `path`.

### Aplicar manifesto em duas etapas

1. Envie a prévia para `POST /api/v1/integracoes/catalogo/aplicacoes/{idAplicacao}/manifestos`.
2. Avalie `diferencas` e guarde o `idManifesto` e `hash` retornados.
3. Aplique a mesma prévia em `POST /api/v1/integracoes/catalogo/aplicacoes/{idAplicacao}/manifestos/{idManifesto}/aplicar`:

```json
{
  "hash": "hash-da-previa"
}
```

4. Exporte o estado ativo com `GET /api/v1/integracoes/catalogo/aplicacoes/{idAplicacao}/manifesto` quando necessário.

A aplicação exige que o `hash` seja exatamente o da prévia pendente. Ao aplicar, permissões e rotas ausentes do novo manifesto são inativadas; itens presentes são criados, atualizados ou reativados. Nunca omita uma permissão ou rota ainda utilizada sem avaliar o impacto nos cargos e acessos existentes.

## Autorização dentro da aplicação

- Proteja endpoints de domínio no backend, não apenas rotas/telas no frontend.
- Autorize usando as chaves em `permissions` retornadas pela sessão ou pelos claims validados no backend.
- Use `capabilities` somente para ações administrativas globais do tenant, como telas de membros e RBAC.
- Uma rota registrada no manifesto declara a permissão requerida, mas a aplicação ainda deve efetivar essa verificação no próprio backend/BFF.
- Trate permissões como chaves estáveis. Renomear a chave no manifesto equivale a criar uma permissão e inativar outra.

## Limitações operacionais atuais

- Não há API de "solicitação de acesso" pelo usuário. Use provisionamento emitido por administrador.
- A Integration API de RBAC não expõe inativação ou reativação de cargo/atribuição. Não chame `PATCH .../status`; esse contrato ainda não existe.
- Token exchange e scopes de gestão delegada dependem da configuração do cliente OIDC. Atualmente, não assuma que todo cliente OIDC recém-criado terá esse grant ou os scopes `sc.*` habilitados.
- Não há OpenAPI publicado para os contratos de integração. Use este skill e valide a integração contra o ambiente da SC.
- A revalidação de sessão BFF não renova a expiração da sessão na SC.

## Checklist de integração

1. Registre a aplicação, seu cliente OIDC, suas URIs de callback e suas origens CORS na SC; obtenha as credenciais BFF e de catálogo e armazene os segredos fora do código-fonte.
2. Declare permissões e rotas no manifesto, crie a prévia e aplique o hash retornado.
3. Modele endpoints e guards usando as chaves de permissão do manifesto.
4. Implemente o BFF e cookie local seguro; não entregue tokens ou segredos ao frontend.
5. Inclua `tenantSubdominio` no login apenas para aplicações não exclusivas; em aplicações exclusivas, deixe a SC resolver o tenant beneficiário configurado. Persista o contexto retornado pela sessão local.
6. Revalide a sessão e encerre a sessão local após `401` da SC.
7. Proteja operações administrativas com capability e permissão no backend.
8. Teste login válido, acesso sem permissão, cargo/atribuição inativos, tenant inativo, contrato/liberação inválidos, logout e rate limit.

## Referências no repositório da SC

- `docs/BFF_INTEGRATION.md`
- `software-center-backend/src/main/java/com/marqf/softwarecenter/controller/CIntegracaoSessaoController.java`
- `software-center-backend/src/main/java/com/marqf/softwarecenter/controller/CIntegracaoRbacController.java`
- `software-center-backend/src/main/java/com/marqf/softwarecenter/controller/CIntegracaoCatalogoController.java`
- `software-center-backend/src/main/java/com/marqf/softwarecenter/dto/RIntegracaoSessaoDtos.java`
- `software-center-backend/src/main/java/com/marqf/softwarecenter/dto/RManifestoDtos.java`
