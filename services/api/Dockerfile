FROM node:18-slim AS base

RUN npm install -g pnpm@8.15.4

WORKDIR /app

# Copy workspace configuration
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY tsconfig.base.json ./

# Copy all packages at once (让 Docker 自己处理)
COPY packages ./packages
COPY infra ./infra
COPY services/api ./services/api

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build stage
FROM base AS builder

WORKDIR /app

# Build all dependencies
RUN pnpm --filter @take-a-break/config build || true
RUN pnpm --filter @take-a-break/firebase build || true  
RUN pnpm --filter @take-a-break/map build || true

# Build API
RUN pnpm --filter @take-a-break/api build

# List output for debugging
RUN find services/api/dist -type f -name "*.js" | head -20

# Production stage
FROM node:18-slim AS production

RUN npm install -g pnpm@8.15.4

WORKDIR /app

# Copy workspace files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY tsconfig.base.json ./

# Copy built artifacts
COPY --from=builder /app/packages ./packages
COPY --from=builder /app/infra ./infra
COPY --from=builder /app/services/api ./services/api

# Install production dependencies only
RUN pnpm install --frozen-lockfile --prod

WORKDIR /app/services/api

EXPOSE 3333

CMD ["node", "dist/services/api/src/server.js"]
