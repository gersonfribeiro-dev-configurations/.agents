# 🤖 Agents & Skills Configuration

Este repositório é o "cérebro" comportamental do assistente OpenCode. Ele define não apenas *como* a IA deve se comunicar, mas *quais* especialidades ela assume dependendo do projeto em que está trabalhando.

A arquitetura é dividida em duas camadas: **Diretrizes Globais** (A Constituição) e **Skills Modulares** (As Especializações).

---

## 🏗️ Arquitetura do Ecossistema

### 1. A Constituição (`AGENTS.md`)
O arquivo `AGENTS.md` funciona como a camada de governança global. Tudo o que estiver aqui é mandatório para qualquer tarefa, independentemente da tecnologia. Ele define:
- **Comunicação:** Idioma, tom de voz e clareza.
- **Padrões de Código:** Nomenclatura (PascalCase, camelCase), tipagem forte e documentação.
- **Arquitetura:** Separação de responsabilidades (Services, Controllers, Utils) e princípios como DRY.

### 2. Skills Modulares (`skills/`)
As Skills são "módulos de conhecimento" que a IA carrega sob demanda. Em vez de sobrecarregar o contexto da IA com todas as regras de todas as linguagens o tempo todo, o OpenCode ativa apenas a skill necessária para o contexto atual.

**Como funciona o ajuste de contexto:**
1. O assistente analisa a tarefa (ex: "Crie um endpoint em Java").
2. Ele identifica a keyword `backend` ou `Java` e carrega a skill correspondente em `skills/backend/SKILL.md`.
3. As instruções da skill são fundidas com as diretrizes globais do `AGENTS.md`.
4. A IA agora opera como um especialista naquela tecnologia específica, seguindo os protocolos de trabalho e checklists de conclusão daquela skill.

---

## 🛠️ Como Adicionar ou Modificar uma Skill

Se você deseja adicionar suporte a uma nova tecnologia (ex: Python/FastAPI), siga este padrão:

1. **Crie a Pasta:** `skills/python-fastapi/`
2. **Crie o arquivo `SKILL.md`:** Use o formato YAML no topo para metadados:
   ```markdown
   ---
   name: python-fastapi
   description: Use quando estiver trabalhando com Python, FastAPI, Pydantic e SQLAlchemy.
   ---
   # Skill: Python FastAPI
   ... (instruções detalhadas)
   ```
3. **Defina o Protocolo:**
   - **Papel:** Quem a IA deve ser.
   - **Protocolo de Trabalho:** Passos obrigatórios (ex: ler documentação via Context7).
   - **Convenções:** Padrões de nomenclatura e estilo de código.
   - **Checklist de Conclusão:** O que deve ser verificado antes de entregar a tarefa.
4. **Atualize o `AGENTS.md`:** Adicione a nova skill na tabela de "Skills disponíveis" para que o assistente saiba que ela existe.

---

## 🔐 Segurança e Privacidade

Este repositório contém **estritamente instruções e padrões**. 
- ❌ **Nunca** adicione chaves de API, senhas ou tokens aqui.
- ❌ **Nunca** coloque caminhos absolutos de pastas pessoais (use `~/` ou variáveis de ambiente).
- ✅ Use referências a variáveis como `{env:MINHA_CHAVE}` se precisar mencionar a existência de um segredo.

## 🚀 Fluxo de Execução

`Tarefa do Usuário` $\rightarrow$ `Análise de Contexto` $\rightarrow$ `Ativação de Skill` $\rightarrow$ `Aplicação de Regras Globais` $\rightarrow$ `Execução Técnica` $\rightarrow$ `Verificação via Checklist` $\rightarrow$ `Entrega`
