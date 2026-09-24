---
  name: packages
  description: Use essa skill sempre que solicitado sobre a criação e manutenção de pacotes próprios do Maven ou do npm.
---

# Instruções de como criar um pacote com o npm no repositório

Aqui, o objetivo é criar uma biblioteca (por exemplo, contendo wrappers do Vuetify ou componentes de manipulação de câmera) que seja instalada via *npm install*.

## Credenciais

Para GitHub Packages privado, usar a credencial autorizada pelo ambiente: `GITHUB_TOKEN` na CI quando o pacote estiver acessível, ou PAT com `read:packages`/`write:packages` conforme operação. Nunca commitar ou imprimir um PAT.

### .npmrc

No projeto, versionar apenas o registry do escopo. A autenticação deve ser fornecida por configuração do usuário/CI protegida, não por token literal no `.npmrc` versionado.

```txt
@meu_usuario:registry=https://npm.pkg.github.com
```

### Preparando o package.json

O GitHub exige que o nome do pacote contenha o escopo de usuário. O arquivo precisa de duas configurações vitais: o nome exato e o publishConfig.

```json
{
  "name": "@meu_usuario/ui-components",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  },
  "scripts": {
    "build": "vite build"
  }
}
```

### Configurações para o vite

O Vite em Library Mode não gera os arquivos de tipagem (.d.ts) automaticamente para os componentes Vue.

Se não gerar esses arquivos, quando for importar o <AppButton> em outro projeto, a IDE (VS Code) vai reclamar que o módulo não tem tipos declarados e perderá o autocomplete das props e emits.

Para resolver isso, só precisa adicionar um plugin no seu vite.config.ts:

- Instale o plugin: npm install -D vite-plugin-dts

#### Adicionar no vite.config.ts:

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import dts from 'vite-plugin-dts' // Importe o plugin
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    vue(),
    dts({ insertTypesEntry: true }) // Adicione o plugin aqui
  ],
  build: {
    // ... restante da configuração ...
  }
})
```

## Versionamento Limpo

Escolher a ferramenta de versionamento que o consumidor realmente utiliza: `standard-version` em um pacote Node, Changesets em workspace/monorepo, ou outra estratégia existente. Conventional Commits/commitlint são aplicáveis quando fazem parte do contrato escolhido.

### A Versão

`standard-version`, quando configurado, calcula bump com base no histórico relevante de Conventional Commits e nas regras da ferramenta (incluindo configuração do projeto); **não** assume sempre MINOR nem apenas o último commit. Ele pode atualizar versão/changelog e criar tag; executar apenas na etapa autorizada do fluxo do consumidor, sem gerar tag no PR de prévia. Changesets/Turbo possuem preparação de mudanças por pacote e publicação própria; consultar o perfil real antes de automatizar.

### Publicação

Após validar a build, publicar somente no fluxo pós-merge aprovado/homologado e com credencial adequada. Conferir versão/tag imutáveis e artefatos que serão enviados antes de executar `npm publish`.


## Consumindo o pacote em outro PWA/Aplicação

Basta configurar o registry de escopo e a autenticação segura no consumidor, e rodar:

```bash
npm install @meu_usuario/ui-components
```

Depois é só importa no código Vue normalmente:

```bash
import { CameraWrapper } from '@meu_usuario/ui-components';
```

---

# Instruções de como criar um pacote com o Maven no repositório

O objetivo aqui é criar um .jar com classes utilitárias, filtros, DTOs e anotações customizadas, para que os microsserviços apenas importem a dependência.

## Credenciais

Para GitHub Packages Maven privado, fornecer credenciais com permissões necessárias no `settings.xml` **fora do repositório** ou via segredo da CI; `GITHUB_TOKEN` pode ser usado quando houver autorização. Não armazenar token literal em exemplos versionados.

### settings.xml

O Maven precisa saber quem eu sou... Devo Editar ou criar o arquivo ~/.m2/settings.xml (no Windows fica em C:\Users\SeuUsuario\.m2\settings.xml).

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>MEU_USUARIO_DO_GITHUB</username>
      <password>${env.GITHUB_PACKAGES_TOKEN}</password>
    </server>
  </servers>
</settings>
```

### Preparando o pom.xml

No repositório do pacote Java, o pom.xml precisa apontar para o GitHub Packages. O elemento <distributionManagement> é o que diz ao Maven para onde enviar o artefato compilado.

```xml
<project>
  <groupId>br.com.meusistema</groupId>
  <artifactId>core-utils</artifactId>
  <version>1.0.0-SNAPSHOT</version> <!-- O JGitver vai manipular isso dinamicamente -->

  <!-- Configuração de Distribuição -->
  <distributionManagement>
    <repository>
      <id>github</id> <!-- Este ID deve ser o mesmo do settings.xml -->
      <name>GitHub Packages</name>
      <url>https://maven.pkg.github.com/MEU_USUARIO/NOME_DO_REPOSITORIO</url>
    </repository>
  </distributionManagement>
</project>
```

## Versionamento Limpo

Técnicas como o uso correto do jgitver e commitlint (.git/hooks para substituir o papel do husky em aplicações frontend) precisam estar no repositório para garantir que as alterações em pacotes vão ser feitas corretamente, como manter v.1.0.0 -> v.1.0.1 e a versão com marcação latest.

### A Versão

Quando adotado, o jgitver calcula versões Maven a partir da configuração e das referências Git, sem exigir edição manual de `<version>` no `pom.xml`. A versão da milestone não substitui automaticamente a do artefato; alinhar tag, build e GitHub Release ao commit integrado.
O maven-antrun-plugin pode ser usado nesse momento para gerar arquivos de properties ou documentação com a versão recém-calculada.

## A Publicação

No terminal

```bash
mvn deploy
```

O Maven compila e publica com as credenciais do `settings.xml` seguro. Executar deploy somente após gates da release; para projetos Go, usar o adaptador Go configurado pelo consumidor e publicar tags somente no fluxo pós-merge.

## Consumindo o pacote em outra API

Na API principal, adicionar a dependência normalmente e indica ao Maven onde procurar:

```xml
<repositories>
  <repository>
    <id>github</id>
    <url>https://maven.pkg.github.com/MEU_USUARIO/NOME_DO_REPOSITORIO</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>br.com.meusistema</groupId>
    <artifactId>core-utils</artifactId>
    <version>1.0.0</version> <!-- Versão gerada pelo JGitver -->
  </dependency>
</dependencies>
```
