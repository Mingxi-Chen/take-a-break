FROM node:18-slim AS base

# Install pnpm
RUN npm install -g pnpm@8.15.4

WORKDIR /app

# Copy workspace files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY tsconfig.base.json ./

# Copy all necessary packages
COPY packages/types ./packages/types
COPY packages/config ./packages/config
COPY packages/map ./packages/map
COPY infra/firebase ./infra/firebase

# Copy API service
COPY services/api ./services/api

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build packages in dependency order
FROM base AS builder

# Build types package (no build needed, just TypeScript definitions)
WORKDIR /app/packages/types

# Build config package
WORKDIR /app/packages/config
RUN pnpm build

# Build firebase package
WORKDIR /app/infra/firebase
RUN pnpm build

# Build map package
WORKDIR /app/packages/map
RUN pnpm build

# Build API service
WORKDIR /app/services/api
RUN pnpm build

# Verify build output
RUN echo "=== Build output ===" && \
    find dist -name "*.js" | head -20

# Production stage
FROM node:18-slim AS production

# Install pnpm
RUN npm install -g pnpm@8.15.4

WORKDIR /app

# Copy package files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY tsconfig.base.json ./

# Copy built packages
COPY --from=builder /app/packages ./packages
COPY --from=builder /app/infra ./infra
COPY --from=builder /app/services/api ./services/api

# Install production dependencies only
RUN pnpm install --frozen-lockfile --prod

WORKDIR /app/services/api

# Expose port
EXPOSE 3333

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD node -e "require('http').get('http://localhost:3333/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start server
CMD ["node", "dist/services/api/src/server.js"]
