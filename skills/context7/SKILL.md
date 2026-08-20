---
name: context7
description: Use when you need to fetch, read, or validate up-to-date documentation for libraries, frameworks, SDKs, or APIs (e.g., Vue, Vuetify, Tailwind).
---

# Skill: Consultas à Documentação (Context7)

Você tem acesso ao Context7, uma ferramenta vital para buscar as documentações oficiais mais atualizadas em vez de depender exclusivamente dos seus dados de treinamento. Isso evita "alucinações", geração de código legado ou sintaxes obsoletas.

## 1. Quando Utilizar Esta Skill

Ative esta skill sempre que o usuário:
- Fizer perguntas de setup ou configuração ("Como configuro o middleware do Next.js?").
- Solicitar geração de código envolvendo bibliotecas ("Escreva uma query do Prisma para...").
- Precisar de referências de API ou validação de propriedades ("Quais são os métodos de auth do Supabase?" ou "Essa prop existe no Vuetify?").
- Mencionar frameworks ou ecossistemas específicos (React, Vue, Svelte, Express, Tailwind, etc.).

## 2. Fluxo Obrigatório de Busca

### Passo 1: Descobrir o ID da Biblioteca (`resolve-library-id`)
Sempre comece chamando a ação `resolve-library-id` com:
- `libraryName`: O nome da tecnologia extraído da solicitação do usuário (ex: "vuetify").
- `query`: A pergunta completa do usuário (isso melhora o algoritmo de ranqueamento de relevância).

### Passo 2: Selecionar o Melhor Match
Escolha o melhor resultado retornado avaliando:
- A correspondência exata ou mais próxima do nome solicitado.
- **Fontes Oficiais:** Prefira pacotes oficiais/primários sobre forks da comunidade (notas de benchmark mais altas indicam melhor qualidade).
- **Atenção à Versão:** Se o usuário mencionar uma versão (ex: "React 19", "Next.js 15"), prefira IDs específicos dessa versão, se disponíveis.

### Passo 3: Buscar a Documentação (`query-docs`)
Utilize a ação `query-docs` informando o `libraryId` selecionado (ex: `/vercel/next.js`) e a sua dúvida.
- Mantenha a busca focada em um **único conceito ou componente por vez** (ex: "v-data-table server side pagination").
- Se a pergunta do usuário abranger múltiplos conceitos distintos (ex: rotas E autenticação E cache), faça chamadas separadas ao `query-docs` para o mesmo ID da biblioteca, a menos que a pergunta seja especificamente sobre como os conceitos interagem.

### Passo 4: Implementação Direta
Incorpore a documentação obtida na sua resposta:
- Responda à dúvida utilizando as informações precisas e atuais.
- Utilize os snippets e exemplos práticos trazidos pela documentação para embasar sua solução de código.
- Cite a versão da biblioteca na resposta quando for relevante.

## 3. Regras de Ouro

- **Priorize a Fonte da Verdade:** Nunca invente propriedades para componentes (especialmente em frameworks como Vuetify ou Vue). Se você não tem 100% de certeza se uma `prop` ou método existe, consulte o Context7 **antes** de gerar o código.
- **Não Misture Consultas:** Consultas combinadas (multi-tópicos no mesmo `query-docs`) diluem o ranqueamento e retornam resultados superficiais. Divida a busca se necessário.
