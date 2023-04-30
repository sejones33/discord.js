FROM node:18-alpine AS builder

RUN apk update
RUN apk add --no-cache libc6-compat
RUN npm i -g pnpm@8.3.1

WORKDIR /usr/proxy

COPY . .
RUN pnpm dlx turbo prune --scope="@discordjs/proxy-container" --docker

FROM node:18-alpine AS installer

RUN apk add --no-cache libc6-compat
RUN apk update
RUN npm i -g pnpm@8.3.1

WORKDIR /usr/proxy
 
COPY .gitignore .gitignore
COPY tsconfig.json tsconfig.json
COPY tsup.config.ts tsup.config.ts
COPY --from=builder /usr/proxy/out/json/ .
COPY --from=builder /usr/proxy/out/pnpm-lock.yaml ./pnpm-lock.yaml
RUN pnpm install
 
COPY --from=builder /usr/proxy/out/full/ .
RUN pnpm dlx turbo run build --filter="proxy-container..."

FROM node:18-alpine AS runner

WORKDIR /usr/proxy

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 proxy
USER proxy

COPY --from=installer --chown=proxy:nodejs /usr/proxy/packages/proxy-container .

CMD ["node", "--enable-source-maps", "dist/index.js"]
