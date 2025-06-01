ARG NODE_VERSION=20
ARG N8N_VERSION=snapshot

# 1. Create an image to build n8n
FROM --platform=linux/arm/v7 koopernic/n8n-base-armv7:${NODE_VERSION} AS builder

WORKDIR /src
COPY . /src

# Use cache mounts for pnpm install
RUN --mount=type=cache,id=pnpm-store,target=/root/.local/share/pnpm/store \
    --mount=type=cache,id=pnpm-metadata,target=/root/.cache/pnpm/metadata \
    DOCKER_BUILD=true pnpm install --frozen-lockfile

RUN pnpm build

# Cleanup unnecessary data
RUN jq 'del(.pnpm.patchedDependencies)' package.json > package.json.tmp && mv package.json.tmp package.json
RUN node .github/scripts/trim-fe-packageJson.js
RUN find . -type f \( -name "*.ts" -o -name "*.vue" -o -name "tsconfig.json" -o -name "*.tsbuildinfo" \) -exec rm -f {} +

# Deploy built app
RUN mkdir /compiled
RUN NODE_ENV=production DOCKER_BUILD=true pnpm --filter=n8n --prod --no-optional --legacy deploy /compiled

# 2. Final runtime image
FROM koopernic/n8n-base-armv7:${NODE_VERSION}
ENV NODE_ENV=production

ARG N8N_RELEASE_TYPE=dev
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}

LABEL org.opencontainers.image.title="n8n"
LABEL org.opencontainers.image.description="Workflow Automation Tool"
LABEL org.opencontainers.image.source="https://github.com/n8n-io/n8n"
LABEL org.opencontainers.image.url="https://n8n.io"
LABEL org.opencontainers.image.version=${N8N_VERSION}

WORKDIR /home/node

COPY --from=builder /compiled /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /

# Optional: Set up task-runner if you have an ARMv7 binary available.
# NOTE: n8n does not currently provide task-runner binaries for ARMv7.
# If you have built your own, you can use this section; otherwise, skip it.
# COPY docker/images/n8n/n8n-task-runners.json /etc/n8n-task-runners.json
# COPY your-own-built-armv7-task-runner /usr/local/bin/task-runner-launcher

# Rebuild native modules and symlink binary
RUN cd /usr/local/lib/node_modules/n8n && \
    npm rebuild sqlite3 && \
    ln -s /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    mkdir /home/node/.n8n && \
    chown -R node:node /home/node/.n8n

ENV SHELL=/bin/sh
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
