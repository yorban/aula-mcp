# syntax=docker/dockerfile:1

#
# Build stage
#
FROM node:24-bookworm-slim AS builder

ARG AULA_MCP_REF=main

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN corepack enable \
    && corepack prepare pnpm@11.1.3 --activate

WORKDIR /app

RUN git clone --depth 1 --branch "${AULA_MCP_REF}" \
    https://github.com/Casperjuel/aula-mcp.git .

# Install only production dependencies needed by the MCP server
# and its workspace dependencies.
RUN pnpm --filter @aula-mcp/mcp-server... \
    install \
    --prod \
    --frozen-lockfile


#
# Runtime stage
#
FROM debian:bookworm-slim AS runtime

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        jq \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Bun is required at runtime, but Node/pnpm/Git are not.
COPY --from=oven/bun:1.3-debian \
    /usr/local/bin/bun \
    /usr/local/bin/bun

WORKDIR /app

# pnpm's workspace links rely on the root node_modules/.pnpm store.
COPY --from=builder /app/node_modules ./node_modules

# Copy only the packages; don't copy CLI, docs, git history, etc.
COPY --from=builder /app/packages ./packages

COPY run.sh /run.sh
RUN chmod +x /run.sh

EXPOSE 7878

CMD ["/run.sh"]
