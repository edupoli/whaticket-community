# FROM node:22-alpine as build-deps

# WORKDIR /usr/src/app

# # Instala dependências de build necessárias primeiro
# RUN apk add --no-cache python3 make g++

# # Cache de dependências
# COPY package*.json ./
# RUN npm install --legacy-peer-deps

# # Configurações de ambiente (após a instalação)
# ENV NODE_OPTIONS=--openssl-legacy-provider
# ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

# # Copia o restante dos arquivos
# COPY .env* ./
# COPY src/ ./src/
# COPY public/ ./public/

# # Build da aplicação
# RUN npm run build

# FROM nginx:alpine

# # Instala ferramentas necessárias
# RUN apk add --no-cache jq openssl curl

# # Instala dockerize
# ENV DOCKERIZE_VERSION v0.6.1
# RUN curl -L https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz | tar -C /usr/local/bin -xzv

# # Configurações do Nginx
# COPY .docker/nginx /etc/nginx/
# COPY --from=build-deps /usr/src/app/build /usr/share/nginx/html

# # Script para variáveis de ambiente
# COPY .docker/add-env-vars.sh /docker-entrypoint.d/01-add-env-vars.sh
# RUN chmod +x /docker-entrypoint.d/01-add-env-vars.sh

# EXPOSE 80

# CMD ["nginx", "-g", "daemon off;"]

FROM node:22-alpine as builder

# Configura o corepack para pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# Configura OpenSSL legacy provider
ENV NODE_OPTIONS=--openssl-legacy-provider

WORKDIR /app

# Cache de dependências
COPY pnpm-lock.yaml ./
RUN pnpm fetch

COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm build

# Estágio de produção
FROM node:22-alpine

WORKDIR /app
COPY --from=builder /app/build ./build

# Instala o serve globalmente
RUN npm install -g serve

EXPOSE 3000
CMD ["serve", "-s", "build", "-l", "3000"]