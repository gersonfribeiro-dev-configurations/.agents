---
name: cicd
description: Use when creating or maintaining CI/CD pipelines, writing Dockerfiles, configuring environments, or setting up deployment automated workflows.
---

# Skill: Pipelines e Entregas (CI/CD)

Você atua na engenharia de confiabilidade e entrega. A infraestrutura e as pipelines devem ser tratadas como código (IaC), sendo reprodutíveis, seguras e eficientes.

## 1. Princípios de Pipeline
Para mudanças de código, considerar as verificações disponíveis na stack; para documentação, templates e infraestrutura, validar seus contratos e cenários pertinentes:
1. **Lint e Formatação:** O código quebra alguma regra do ESLint/Prettier ou equivalente?
2. **Testes Automatizados:** Execução das suítes de teste (Unitários, Integração E2E).
3. **Análise Estática:** SonarQube quando adotado, com gate em PR e check obrigatório quando configurado.
4. **Build:** Compilação/empacotamento quando a mudança produzir artefatos executáveis.

## Workflows reutilizáveis e versionamento

- Descrever contrato `workflow_call` (entradas, saídas, permissões, segredos e versões) e fornecer callers curtos com referência estável revisada; não acoplar o workflow à organização ou versão fixa.
- Prévia de PR opera com leitura somente, sem publicar tag/release ou permitir segredo de escrita em código de PR. Publicar somente após merge, review e homologação exigidos pelo consumidor; conferir SHA/branch integrada e privilégio mínimo (`contents: write` apenas nesse fluxo).
- Milestone da sprint não determina a versão da aplicação: usar o adaptador compatível com o consumidor (`standard-version`, Changesets/Turbo, Maven/jgitver, Go/go-gitsemver conforme presente). Preservar tags publicadas, detectar concorrência e reexecução; nunca mover tag para encobrir divergência.

## 2. Conteinerização (Docker)
- **Dockerfiles Otimizados:** Utilize builds multi-stage para separar o ambiente de compilação do ambiente de runtime. A imagem final deve conter apenas o necessário para rodar a aplicação.
- **Segurança:** Não rode aplicações como `root` dentro do container a menos que seja estritamente necessário. Defina usuários sem privilégios.
- Utilize cache de camadas de forma inteligente (copie arquivos de configuração de dependências, como `package.json` ou `pom.xml`, antes do código fonte).

## 3. Ambientes (Staging / Prod)
- Variáveis que alteram comportamento por ambiente devem ser parametrizadas via segredos (Secrets) ou injetadas na pipeline. NUNCA faça commit de `.env` com dados reais.
- Mantenha os scripts de deploy idempotentes (podem ser executados múltiplas vezes sem causar estado inconsistente).
