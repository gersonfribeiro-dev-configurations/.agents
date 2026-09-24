---
name: toolkit
description: Use when you need to interact with the Dockerized environment, execute CLI commands, manage services, or understand the infrastructure boundaries.
---

# Skill: Operações de Infraestrutura Local (Toolkit / MCP_DOCKER)

Primeiro identifique se esta sessão roda no host, em container ou por sandbox MCP. Esta skill orienta rede e serviços locais conforme o ambiente realmente detectado.

## 1. Regras de Rede (Networking)
- **Acesso ao Host:** Dentro de container, `localhost` aponta para o próprio container; `host.docker.internal` pode permitir acesso ao host quando resolvido/configurado. Em sessão no host, `localhost` aponta para o próprio host. Testar conectividade antes de escolher endereço.
- **Exemplo de Acesso:** Se o frontend roda na porta 5173 do host, acesse via `http://host.docker.internal:5173`.

## 2. Execução de Comandos
- Antes de executar scripts complexos ou instalar pacotes pesados no ambiente, certifique-se de que está no diretório correto (`pwd`).
- Ao ler logs de containers ou saídas de terminal, limite a leitura aos últimos dados relevantes para não sobrecarregar o contexto da conversa.

## 3. Segurança e Limites
- Não altere configurações de rede ou derrube containers de banco de dados a menos que explicitamente solicitado pelo usuário para fins de reset de ambiente.

## 4. Ciclo de Vida de Sandboxes MCP
- Ferramentas MCP que criam sandboxes persistentes podem deixar containers em execução quando a chamada de limpeza não ocorre. Antes de atribuir containers extras ao projeto, inspecione imagem, labels e mounts com `docker inspect` e `docker ps --format`.
- Containers com o label `docker-mcp-name=node-code-sandbox` pertencem ao sandbox Node do MCP, não ao `docker-compose` da aplicação. Eles normalmente têm nomes aleatórios, usam uma imagem Node e montam o socket Docker em `/var/run/docker.sock`.
- Ao usar `MCP_DOCKER_sandbox_initialize`, encerre obrigatoriamente o sandbox ao final da tarefa com `MCP_DOCKER_sandbox_stop`, inclusive quando houver erro durante a execução. Para comandos pontuais, prefira `MCP_DOCKER_run_js_ephemeral`, que remove o container automaticamente.
- Para remover apenas sandboxes órfãos, nunca use limpeza global de containers. Filtre pelo label: no PowerShell, `docker ps -aq --filter "label=docker-mcp-name=node-code-sandbox" | ForEach-Object { docker rm -f $_ }`.

## 5. Comunicação Inter-serviços
- Se você receber erros de "health-check" ou "backend inalcançável" ao testar o frontend, verifique se o backend faz parte da mesma rede Docker.
- Caso estejam no mesmo `docker-compose`, não use `host.docker.internal` para a comunicação entre eles. Use o nome do serviço definido no compose (ex: `http://backend:8080/health`).
