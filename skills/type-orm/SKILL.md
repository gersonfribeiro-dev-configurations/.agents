---
name: type-orm
description: Use when working with TypeORM in Node.js or TypeScript applications, managing relational databases, defining entities and relations, executing migrations, handling transactions, and optimizing database operations using Repositories and QueryBuilder.
---

# Diretrizes da Skill: TypeORMEsta skill orienta o uso do TypeORM em aplicações Node.js/TypeScript, focando em performance, manutenibilidade e nas melhores práticas da ferramenta.

1. Padrões de Arquitetura: Data Mapper vs Active Record

## O TypeORM suporta dois padrões principais. A escolha deve se basear na complexidade da aplicação:

### Data Mapper (Recomendado para aplicações médias/grandes):

Separa a lógica de negócio do banco de dados. Todas as operações são feitas através de um Repository. Garante um código mais limpo e testável.

### Active Record (Recomendado para apps simples/MVPs):

Os métodos de banco de dados ficam na própria entidade (estendendo BaseEntity). Facilita a prototipagem rápida, mas pode ferir o princípio da responsabilidade única (SRP) conforme a aplicação cresce.

2. Modelagem de Entidades (Entities)

## Sempre defina suas entidades utilizando os decorators oficiais e aproveite a tipagem estática do TypeScript.

- Use @Entity() para definir a tabela.
- Use @PrimaryGeneratedColumn('uuid') para chaves primárias seguras (evita expor o tamanho do banco para web scrapers).
- Seja explícito com tipos e restrições:

```typescript
@Column({ type: 'varchar', length: 100, nullable: false }).
```

- Para propriedades de data automáticas, use @CreateDateColumn() e @UpdateDateColumn().

3. Performance e Operações de Banco

## O perigo do .save()

Evite usar o método repository.save() cegamente. O save() tem uma dupla função: se a entidade não existe, ele insere; se existe, ele atualiza. Para isso, ele sempre executa uma instrução SELECT antes do INSERT ou UPDATE, o que gera custo duplo no banco de dados.

- Para inserir: Use repository.insert() quando souber que o registro é novo.
- Para atualizar: Use repository.update() passando o ID ou a condição.
- Inserções em Massa (Bulk Inserts)
- Para salvar grandes arrays, evite gargalos de memória dividindo a operação em chunks:

```typescript
await repository.save(arrayWithThousandsOfItems, { chunk: 500 });
```

## Paginação

Sempre utilize skip e take no lugar de trazer todos os dados para a memória. O método findAndCount() é o ideal para retornar itens e o total de páginas de forma otimizada.

4. Transações Seguras (Transactions)

## A regra de ouro do TypeORM para transações:

### *NUNCA* utilize o Entity Manager ou Repositories globais dentro de uma transação.

Você deve sempre usar o transactionalEntityManager fornecido no callback, caso contrário, suas queries rodarão fora do escopo da transação (podendo causar inconsistências graves).

```typescript
await dataSource.transaction(async (transactionalEntityManager) => {
  // CORRETO: Usando o manager da transação
  await transactionalEntityManager.insert(User, { name: 'João' });
  
  // Para usar Repositórios customizados dentro da transação:
  const customRepo = transactionalEntityManager.withRepository(UserRepository);
  await customRepo.doSomething();
});
```

5. Consultas Complexas (QueryBuilder)

Para consultas que envolvam múltiplos JOINs, subqueries ou funções específicas do banco de dados, abandone os métodos genéricos (find, findOne) e utilize o QueryBuilder.

## Cuidado com SQL Injection:

NUNCA use interpolação de strings (${variavel}) em cláusulas .where(). Sempre utilize parameterized queries:

```typescript
// ❌ ERRADO (Vulnerável)
.where(`user.name = '${userName}'`) 

// ✅ CORRETO (Seguro)
.where("user.name = :name", { name: userName })
```

6. Configurações de Produção e Migrations

## synchronize: true:

### *NUNCA* utilize em produção. 

Essa flag sincroniza as entidades com o banco de forma automática e pode dropar tabelas ou colunas acidentalmente.
Mantenha habilitado apenas no ambiente de dev ou utilize Migrations desde o início.

### Migrations:

- Centralize as alterações do esquema de banco de dados em migrations geradas pelo CLI do TypeORM (typeorm migration:generate).
- Revise o código SQL gerado automaticamente antes de aplicar (typeorm migration:run).
- Naming Strategy: Considere usar uma estratégia de nomenclatura (como a SnakeNamingStrategy) nas conexões para que propriedades em camelCase no código TypeScript se transformem automaticamente em snake_case no banco (ex: createdAt -> created_at), mantendo o banco legível para DBAs.
