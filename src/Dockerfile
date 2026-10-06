# --- Этап 1: Сборка приложения и установка ВСЕХ зависимостей ---
FROM node:20-bookworm-slim AS builder
WORKDIR /app

# Устанавливаем системные зависимости для сборки бинарников (better-sqlite3)
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
RUN npm ci

# Копируем исходный код
COPY . .

ENV NEXT_TELEMETRY_DISABLED=1
# Собираем продакшен-билд (next build)
RUN npm run build

# ВАЖНО: Удаляем все devDependencies (TS, ESLint, Tailwind CLI и т.д.), 
# оставляя только бинарники и продакшен-пакеты.
RUN npm prune --production

# --- Этап 2: Финальный легковесный продакшен-образ ---
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

# Копируем очищенные node_modules, сборку .next, package.json и абсолютно все скрипты
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/scripts ./scripts

# Копируем конфигурационные файлы (часто проверяются кастомными скриптами запуска)
COPY --from=builder --chown=nextjs:nodejs /app/next.config.* ./
COPY --from=builder --chown=nextjs:nodejs /app/tsconfig*.json ./

USER nextjs

EXPOSE 8008

# Возвращаем оригинальную команду запуска, которая работала в самом начале
CMD ["npm", "run", "start"]
