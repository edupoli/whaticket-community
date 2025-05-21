FROM node:18-bullseye as build-deps

# Instala dependências do sistema
RUN apt-get update && \
  apt-get install -y --no-install-recommends \
  wget \
  gnupg && \
  rm -rf /var/lib/apt/lists/*

# Instala dockerize e dumb-init
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
  tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
  rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

WORKDIR /usr/src/app

# Instala pnpm
RUN npm install -g pnpm@latest

# Copia e instala dependências
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod
RUN pnpm add -D typescript
# Configurações do ambiente
ENV NODE_OPTIONS=--openssl-legacy-provider
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV CHROME_BIN=/usr/bin/google-chrome-stable

# Copia o restante dos arquivos
COPY . .

# Build da aplicação
RUN pnpm run build

# Configurações finais
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

ENTRYPOINT ["dumb-init", "--"]
CMD dockerize -wait tcp://${DB_HOST}:3306 -timeout 60s && \
  pnpm exec sequelize db:migrate && \
  node dist/server.js