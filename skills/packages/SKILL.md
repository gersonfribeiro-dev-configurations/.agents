---
  name: packages
  description: Use essa skill sempre que solicitado sobre a criação e manutenção de pacotes próprios do Maven ou do npm.
---

# Instruções de como criar um pacote com o npm no repositório

Aqui, o objetivo é criar uma biblioteca (por exemplo, contendo wrappers do Vuetify ou componentes de manipulação de câmera) que seja instalada via *npm install*.

## Credenciais

Como nosso repositório de pacotes é sempre privado para restringir o seu uso apenas a nossa organização, temos que configurar um Personal Access Token no Github e adicionar no .npmrc. O PAT utilizado deve conter os escopos de write:packages e read:packages

### .npmrc

Na raiz do projeto da biblioteca, criar um arquivo chamado .npmrc.
Ele instrui o NPM a direcionar o download e o upload de pacotes com o escopo (meu nome de usuário) para o GitHub, usando o token PAT.

```txt
@meu_usuario:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=MEU_PERSONAL_ACCESS_TOKEN
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
    "build": "vite build",
    "release": "standard-version"
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

Técnicas como o uso correto do standard-version e commitlint precisam estar no repositório para garantir que as alterações em pacotes vão ser feitas corretamente, como manter v.1.0.0 -> v.1.0.1 e a versão com marcação latest.

### A Versão

Ao rodar o standard-version (npm run release), a ferramenta gera o CHANGELOG.md, cria a tag no Git e atualiza o package.json. Um detalhe comportamental importante a ter em mente sobre o standard-version na configuração é que ele irá incrementar sempre a versão MINOR, independentemente de realizar um commit de fix ou feat, e ele avalia apenas o último commit. Isso significa que a esteira de publicação dará saltos de MINOR a cada nova release (ex: 1.0.0 para 1.1.0, depois 1.2.0), não gerando versões de patch. Para um ecossistema interno de componentes, esse avanço acelerado de MINOR é perfeitamente funcional.

### Publicação

Após rodar o build do Vue/TypeScript, executar *npm publish*. O NPM lê o *.npmrc*, autentica no GitHub e envia os artefatos da pasta *dist/*.


## Consumindo o pacote em outro PWA/Aplicação

Basta garantir que no novo projeto, o .npmrc com a autenticação exista na raiz, e rodar:

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

Como nosso repositório de pacotes é sempre privado para restringir o seu uso apenas a nossa organização, temos que configurar um Personal Access Token no Github e adicionar no settings.xml. O PAT utilizado deve conter os escopos de write:packages e read:packages

### settings.xml

O Maven precisa saber quem eu sou... Devo Editar ou criar o arquivo ~/.m2/settings.xml (no Windows fica em C:\Users\SeuUsuario\.m2\settings.xml).

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>MEU_USUARIO_DO_GITHUB</username>
      <password>MEU_PERSONAL_ACCESS_TOKEN</password>
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

O jgitver entra em ação e calcula a próxima versão com base nas tags e branchs do Git, injetando isso dinamicamente no Maven (não precisa editar a tag <version> no pom.xml manualmente).
O maven-antrun-plugin pode ser usado nesse momento para gerar arquivos de properties ou documentação com a versão recém-calculada.

## A Publicação

No terminal

```bash
mvn deploy
```

O Maven compila o código, empacota em um .jar, lê as credenciais do settings.xml e envia o pacote para o GitHub.

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
