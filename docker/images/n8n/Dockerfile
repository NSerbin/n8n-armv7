ARG NODE_VERSION=20

FROM koopernic/n8n-base-armv7:${NODE_VERSION}

ARG N8N_VERSION=1.46.0
RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable

# Set SHELL option for pipefail
SHELL ["/bin/sh", "-o", "pipefail", "-c"]

RUN set -eux; \
    npm install --no-audit -g --omit=dev n8n@${N8N_VERSION} --ignore-scripts && \
    npm rebuild --prefix=/usr/local/lib/node_modules/n8n sqlite3 && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/chat && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/n8n-design-system && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/node_modules && \
    find /usr/local/lib/node_modules/n8n -type f \( -name "*.ts" -o -name "*.js.map" -o -name "*.vue" \) -print0 | xargs -0 rm -f && \
    rm -rf /root/.npm

COPY ./docker/images/n8n/docker-entrypoint.sh /

RUN \
    mkdir .n8n && \
    chown node:node .n8n

ENV SHELL /bin/sh
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
