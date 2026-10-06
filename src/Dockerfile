FROM node:20-alpine AS builder
WORKDIR /app

RUN apk add --no-cache python3 make g++ \
    && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build \
    && npm prune --omit=dev \
    && npm cache clean --force \
    && rm -rf .next/cache .next/standalone \
    && find .next -name '*.map' -delete \
    && rm -rf node_modules/better-sqlite3/deps node_modules/better-sqlite3/src \
              node_modules/better-sqlite3/build/Release/obj \
              node_modules/better-sqlite3/build/Release/.deps

FROM node:20-bookworm-slim AS runtime
WORKDIR /app

ENV NODE_ENV=production \
    HOST=0.0.0.0 \
    HTTP_PORT=8008 \
    VICKY_RUNTIME_TOPOLOGY=single-process-local \
    NEXT_TELEMETRY_DISABLED=1

COPY --from=builder /app ./

EXPOSE 8008
CMD ["npm", "run", "start"]
