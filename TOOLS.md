# 🛠️ Guide: Tools & Model Context Protocol (MCP)

Este guia explica como funciona a camada de ferramentas do OpenCode. O OpenCode não é apenas um chat; ele é capaz de interagir com o seu sistema operacional, APIs externas e ferramentas de design através do **Model Context Protocol (MCP)**.

## 🧩 O que é MCP?

O **Model Context Protocol** é um padrão que permite que a IA "conecte" ferramentas externas ao seu fluxo de pensamento. Imagine que a IA tem "braços" para alcançar o seu terminal, o seu navegador ou o seu editor de código.

No OpenCode, dividimos as ferramentas em dois tipos:

### 1. MCPs Remotos (Cloud)
São serviços que rodam em servidores externos. A IA envia uma requisição via HTTPS e recebe a resposta.
- **Exemplo: Context7:** Funciona como um "Oráculo" de documentações. Em vez de a IA confiar na memória (que pode estar desatualizada), ela consulta a documentação oficial em tempo real.
- **Exemplo: GitHub:** Permite que a IA gerencie Issues, Pull Requests e analise repositórios diretamente via API.

### 2. MCPs Locais (System)
São scripts ou binários que rodam na sua própria máquina. Eles dão à IA a capacidade de executar comandos reais.
- **Exemplo: Pencil:** Conecta a IA ao software de prototipagem `pen.dev`, permitindo que ela crie interfaces visuais.
- **Exemplo: MCP_DOCKER:** Um gateway que permite que a IA interaja com containers Docker, verifique logs do Grafana/Loki e analise a infraestrutura.

---

## ⚙️ Configuração para Iniciantes

Para fazer as ferramentas funcionarem, você precisa configurar o arquivo `opencode.json` (geralmente em `~/.config/opencode/`).

### Passo a Passo de Instalação:

1. **Variáveis de Ambiente:** 
   A maioria das ferramentas exige chaves de API. **Nunca coloque a chave diretamente no JSON**. Use a sintaxe `{env:NOME_DA_VARIAVEL}`.
   - No Windows: Adicione a variável nas "Variáveis de Ambiente do Sistema".
   - No Linux/Mac: Use um arquivo `.env` ou `direnv`.

2. **Configurando o `opencode.json`:**
   ```json
   "mcp": {
     "nome-da-ferramenta": {
       "type": "remote", // ou "local"
       "url": "https://api.exemplo.com/mcp", // se for remoto
       "command": ["caminho/para/script.bat"], // se for local
       "enabled": true
     }
   }
   ```

3. **O Segredo do `.envrc` (Para Projetos Específicos):**
   Se você trabalha em múltiplos projetos, pode querer que a IA use tokens diferentes para cada um. Crie um arquivo `.envrc` na raiz do projeto:
   ```bash
   export GITHUB_PERSONAL_ACCESS_TOKEN="seu_token_especifico_do_projeto"
   ```
   Isso garante que a IA tenha permissões estritas apenas para aquele repositório.

---

## 🛡️ Segurança e Boas Práticas

- **Princípio do Menor Privilégio:** Ao criar um Token do GitHub (PAT), dê apenas as permissões necessárias (ex: `repo:read`).
- **Ignorar Segredos:** Sempre adicione `.envrc` ou arquivos de tokens ao seu `.gitignore`.
- **Validação de Local:** MCPs locais executam comandos no seu PC. Sempre revise o que a IA está tentando executar se você não confia na origem do script.

## 🚀 Resumo de Fluxo
`IA decide usar ferramenta` $\rightarrow$ `Consulta opencode.json` $\rightarrow$ `Busca chave no Ambiente` $\rightarrow$ `Executa Comando/Requisição` $\rightarrow$ `Recebe Dados` $\rightarrow$ `Processa Resposta para o Usuário`
