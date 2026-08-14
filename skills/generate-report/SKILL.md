---
name: generate-report
description: "Use when you need to generate internal task completion reports with 4 standardized sections: Realização (what was done), Fontes modificados (file paths), p/ teste (testing instructions for non-technical testers), and O que há de novo (changelog notes). Automatically extracts modified files from Git."
argument-hint: "[task description or context]"
---

# Gerador de Relatórios de Tarefas

## O que faz

Gera relatórios estruturados e padronizados para controle interno de tarefas concluídas, facilitando comunicação entre desenvolvimento e teste. Cada relatório contém 4 seções essenciais com informações prontas para documentação e sistemas internos.

## Quando usar

Sempre que você concluir uma tarefa ou feature e precisar documentar:
- ✅ Para repassar para a equipe de testes
- ✅ Para preencher sistemas internos de controle
- ✅ Para gerar change notes do sistema
- ✅ Para documentar modificações sem jargão técnico

## Procedimento

Quando você pedir **"Gere o relatório para o serviço por favor"**, sigo estes passos:

### 1. Extração Automática do Git

- Obtenho os arquivos modificados do último commit/branch
- Listo apenas os paths relativos (sem prefixos técnicos desnecessários)

### 2. Seções do Relatório

#### **Realização (Interno)**

- Descrição breve (3-5 linhas) do que foi implementado
- Foco no resultado, não em detalhes técnicos
- Linguagem clara e direta

#### **Fontes Modificados**

- Lista de caminhos dos arquivos editados
- Um arquivo por linha
- Paths relativos ao projeto

#### **p/ Teste**

- Instruções step-by-step para a equipe acessar a mudança
- Linguagem **simples**, sem jargão técnico (sem "refactor", "merge", "deploy", etc.)
- Como chegar ao recurso modificado (ex: "abra o menu > clique em X")
- Como testar se funciona (ex: "clique no botão Y e veja se Z acontece")
- Suposições mínimas sobre conhecimento técnico

#### **O que há de novo**

- Texto curto (1-3 linhas) para change notes da versão
- Descreve o benefício/mudança visível para usuários
- Tom informal, sem termos técnicos

### 3. Formato

- Markdown estruturado, pronto para copiar/colar
- Seções bem delimitadas
- Fácil de separar e usar em diferentes sistemas

## Exemplo de Uso

**Você pede:**

```
Gere o relatório para o serviço por favor
```

**Eu gero:**

```markdown
## Relatório de Tarefa Concluída

### Realização (Interno)
Implementada funcionalidade de filtro por status no painel de tarefas. Agora usuários podem visualizar apenas tarefas ativas, concluídas ou pausadas através de abas interativas.

### Fontes Modificados

- `src/components/TaskPanel.vue`
- `src/stores/taskStore.ts`
- `src/api/tasks.ts`

### p/ Teste

1. Acesse a página de Tarefas (menu esquerdo > Tarefas)
2. No topo da lista, você verá três abas: "Ativas", "Concluídas" e "Pausadas"
3. Clique em cada aba e confirme que só aparecem tarefas do tipo selecionado
4. Crie uma nova tarefa, marque como concluída e veja se desaparece da aba "Ativas"

### O que há de novo

Novo sistema de filtros para tarefas. Organize sua lista por status com um simples clique nas abas.
```

## Princípios

- **Clareza acima de tudo**: A equipe de testes deve entender sem perguntar
- **Sem jargão**: Nada de "deploy", "branch", "refactor", "endpoint", etc.
- **Concisão**: Informação útil, sem fluff
- **Consistência**: Sempre a mesma estrutura, mesma ordem

## Armazenamento

Essas instruções são **persistentes**. Cada vez que você pedir "Gere o relatório para o serviço por favor", serei sempre acionado automaticamente com este guia em contexto.
