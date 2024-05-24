# Relatório da Configuração e Execução do Backstage com Docker

## 1. Instalação do Backstage

**1.1. Acesso ao Diretório:** 
   No terminal, acessar a pasta onde o projeto será criado.

**1.2. Criação do Projeto:** 
   Executar o comando para criar a estrutura inicial do projeto Backstage:
   ```bash
   npx @backstage/create-app@latest --skip-install
   ```

**1.3. Instalação das dependências:**
   Executar o comando abaixo para instalar as dependências necessárias:
   ```bash
   yarn install
   ```
   *Pode ser necessário atualizar o Node ou Yarn.*


## 2. Preparação para Build

**2.1. Instalação de Dependências:**
   Executar o comando para garantir que as dependências sejam instaladas conforme definido no lockfile.
   ```bash
   yarn install --frozen-lockfile
   ```

**2.2. Criação de Types:**
   Compilar os tipos TypeScript do projeto usando:
   ```bash
   yarn tsc
   ```

**2.3. Build da aplicação:**
   Construir o backend do Backstage com:
   ```bash
   yarn build:backend
   ```

## 3. Ajuste do Dockerfile

**3.1 Abertura do Dockerfile:**
   Abrir o Dockerfile localizado em 'packages > backend > Dockerfile' usando o Visual Studio Code.

**3.2 Edição do Dockerfile:**
   Alterar o código do Dockerfile pelo código abaixo:
   ```bash
   FROM node:18-bookworm-slim

# Install isolate-vm dependencies, these are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# From here on we use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will
# fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV development

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
```

## 4. Criação da imagem do Backstage e execução

**4.1 Criação da imagem Docker:**
   Criar a imagem executando o seguinte comando:
   ```bash
   docker image build . -f packages/backend/Dockerfile --tag backstage --no-cache
   ```

**4.2 Execução do container Docker:**
   Executar o container do Backstage usando o comando:
   ```bash
   docker run -it -p 7007:7007 backstage
   ```

**4.3 Acesso ao Backstage:**
   Após a execução do container, acessar 'http://localhost:7007' para utilizar o Catálogo de Serviços do Backstage.

   ![aplicacao rodando}](https://github.com/BeatrizHirasaki/ponderada-backstage/blob/main/assets/aplicacao-rodando.png)

