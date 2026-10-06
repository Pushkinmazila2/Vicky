# --- Этап 1: Установка и сборка зависимостей ---
FROM node:20-bookworm-slim AS deps
WORKDIR /app

# Устанавливаем инструменты сборки (необходимы для компиляции better-sqlite3)
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
RUN npm ci

# --- Этап 2: Сборка Next.js приложения ---
FROM node:20-bookworm-slim AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED=1

RUN npm run build

# --- Этап 3: Финальный продакшен-образ ---
FROM node:20-bookworm-slim AS runtime
WORKDIR /app

ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=8008
ENV HTTP_PORT=8008
ENV VICKY_RUNTIME_TOPOLOGY=single-process-local

# Безопасность: работаем не из-под root
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# 1. Копируем ядро standalone-сервера
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

# 2. ВАЖНО: Вручную копируем скомпилированный better-sqlite3 из папки сборщика.
# Next.js standalone ожидает внешние пакеты в корневом node_modules внутри standalone-папки.
COPY --from=builder --chown=nextjs:nodejs /app/node_modules/better-sqlite3 ./node_modules/better-sqlite3
COPY --from=builder --chown=nextjs:nodejs /app/node_modules/bindings ./node_modules/bindings
COPY --from=builder --chown=nextjs:nodejs /app/node_modules/file-uri-to-path ./node_modules/file-uri-to-path

USER nextjs

EXPOSE 8008

CMD ["node", "server.js"]
