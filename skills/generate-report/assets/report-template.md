# Modelo de Relatório de Tarefa

Use este template como referência para o formato esperado dos relatórios.

---

## Relatório de Tarefa Concluída

**Data:** [YYYY-MM-DD]

**Tarefa:** [Título ou ID da tarefa]

### Realização (Interno)

[Descrição breve do que foi feito em 3-5 linhas. Foco no resultado.]

Exemplo:

> Implementada funcionalidade de filtro por status no painel de tarefas. Agora usuários podem visualizar apenas tarefas ativas, concluídas ou pausadas através de abas interativas. Integrado com o store existente sem quebrar compatibilidade.

### Fontes Modificados

- `src/components/TaskPanel.vue`
- `src/stores/taskStore.ts`
- `src/api/tasks.ts`
- `tests/TaskPanel.spec.ts`

### p/ Teste

1. [Passo 1 - como chegar ao recurso]
2. [Passo 2 - o que fazer]
3. [Passo 3 - o que esperar]
4. [Passo 4 (opcional) - casos adicionais]

Exemplo:

1. Acesse a página de Tarefas (menu esquerdo > Tarefas)
2. No topo da lista, você verá três abas: "Ativas", "Concluídas" e "Pausadas"
3. Clique em cada aba e confirme que só aparecem tarefas do tipo selecionado
4. Crie uma nova tarefa, marque como concluída e veja se desaparece da aba "Ativas" e aparece em "Concluídas"
5. Teste o botão "Limpar filtros" (se existir) para retornar à visualização completa

### O que há de novo

[Texto curto (1-3 linhas) descrevendo o benefício/mudança visível.]

Exemplo:

> Novo sistema de filtros para tarefas. Organize sua lista por status com um simples clique nas abas. Melhora a produtividade ao focar apenas nas tarefas relevantes.

---

## Dicas de Redação

### Para a seção de Realização (Interno)

- ✅ Use linguagem técnica se necessário (é para desenvolvedores/PO)
- ✅ Descreva o **o quê** e **por quê**
- ✅ Mencione integrações importantes
- ❌ Não detalhe *como* foi implementado (nem ninguém se importa com padrões de design aqui)

### Para a seção de p/ Teste

- ✅ Instrua como um guia de usabilidade
- ✅ Use palavras simples: "clique", "abra", "marque", "veja"
- ✅ Seja específico: "menu esquerdo > Tarefas" (não "navegue")
- ✅ Indique resultado esperado: "você verá...", "vai desaparecer...", "deve aparecer..."
- ❌ Nunca diga: "faça deploy", "rode os testes", "abra o DevTools"
- ❌ Não use: "endpoint", "branch", "refactor", "query", "middleware"

### Para a seção de O que há de novo

- ✅ Foco no benefício do usuário
- ✅ Tone informal, amigável
- ✅ Uma frase principal + resultado prático
- ❌ Não descreva implementação técnica

---

## Checklist Antes de Solicitar o Relatório

- [ ] A tarefa está realmente concluída (não WIP)
- [ ] Os arquivos foram commitados/salvos
- [ ] Você tem clareza sobre o que foi alterado
- [ ] Pode instruir alguém (sem conhecimento técnico) como testar
