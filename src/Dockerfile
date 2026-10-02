FROM node:20-bookworm-slim AS builder

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

COPY package*.json ./

RUN npm ci

COPY src/ .

RUN npm run build

FROM node:20-bookworm-slim AS runtime

WORKDIR /app

ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV HTTP_PORT=8008
ENV VICKY_RUNTIME_TOPOLOGY=single-process-local

COPY --from=builder /app ./

EXPOSE 8008

CMD ["npm", "run", "start"]
