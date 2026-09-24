---
name: vue
description: Use when working with Vue 3 components, Vuetify, Pinia stores, composables, or Vue-specific patterns including template structure, component design, and reactive state management.
---

# Skill: Vue 3 + Vuetify

## Role

Desenvolvedor Frontend especialista em Progressive Web Apps, Vue 3, Vuetify, TypeScript, SCSS e interfaces reativas.

## Escopo e stack

- Aplicar apenas a projetos Vue; confirmar quais tecnologias estão presentes (Composition API, Vuetify, TypeScript, SCSS, PWA, Pinia e Vue Router) antes de exigir padrões específicos.
- Contratos de `BaseForm`, `BaseDialog`, `GenericView`, `CResolvePayloadFiltros` e outras classes próprias do template pertencem à skill `boilerplate-vue`, não a qualquer aplicação Vue.

## Estrutura do `<script setup>`

Se o projeto oferecer o snippet `vsetup` em `.vscode/vue-component.code-snippets`, siga sua ordem:

1. `// Types e Interfaces` com JSDoc `@property`
2. `defineProps` / `defineEmits`
3. `// Constantes`
4. `// Reativas - Model` (`defineModel`)
5. `// Reativas - Ref` (`ref`)
6. `// Computadas`
7. `// Funções`
8. `// Observadores`
9. `// Lifecycle Hooks`
10. `// Expose`

Quando houver apenas um tipo de reatividade, usar `// Reativas` (sem subcategoria).

## Props e emits

`defineProps` e `defineEmits` devem ficar logo abaixo dos seus respectivos types. Props antes de emits. Eventos devem representar intenções de negócio (`save`, `cancel`), não implementação (`clickButton`).

## Componentes

- Priorizar criação de componentes reutilizáveis.
- Componentes Base devem ser genéricos e não conhecer regras de negócio.
- Componentes reutilizáveis devem ter comentário curto de responsabilidade.

## Vuetify

- Priorizar componentes nativos do Vuetify antes de wrappers ou CSS estrutural próprio.
- Wrappers só quando encapsularem comportamento real (loading, permissões, defaults).
- Layout responsivo com `useDisplay`.
- Hotkeys quando houver composable instalado e necessidade de acessibilidade em desktop.

## Layout

- Layout controla estrutura, responsividade e scroll; seguir a arquitetura e o roteamento existentes em cada aplicação.

## SCSS

- Seguir a convenção de SCSS existente; separar estilos reutilizáveis quando trouxer clareza.
- Scrollbars discretas com mixins reutilizáveis.

## Pinia

Stores contêm apenas estado compartilhado: preferências, tema, idioma, layout persistente, filtros globais, paginação compartilhada, micro cache. Não conhecem componentes, não chamam HTTP diretamente, não concentram regras de services.

## Composables

Contêm lógica reutilizável de estado/comportamento: preparar parâmetros, tratar erros, snackbar, redirect, loading, encapsular UX.

Não impor uma cadeia fixa para toda tela: UI orquestra UX, services cuidam de integrações, stores mantêm estado compartilhado.

## Performance

- Preferir `computed` a `watch`.
- Evitar computed encadeadas desnecessárias.
- `key` obrigatória em listas.
- Lazy loading para rotas e componentes pesados.

## Loading e lock

Toda requisição acionada pela UI deve ter lock visual para evitar disparos duplicados.

## PWA

Responsividade nativa, ícones no manifest, atenção a `icon`/`badge` em notificações web push.

## Navegação

- Itens de navegação derivados das rotas e metadados.
- Preferências de usuário em stores Pinia com persistência centralizada.
- Sem acesso direto a `localStorage` em componentes.

## Contratos específicos de aplicação

Para `BaseForm`, `BaseDialog`, `GenericView`, filtros, formatters e gráficos do boilerplate, consulte `boilerplate-vue` e confirme a versão desses componentes no consumidor antes de adotar os contratos.
