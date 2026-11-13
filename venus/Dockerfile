# VenusVerse image: builds outline fully from this fork's source.
# Merge of upstream Dockerfile.base (deps+build) + Dockerfile (runner),
# so we do not depend on the published outlinewiki/outline-base image.

ARG APP_PATH=/opt/outline

# --- deps + build (from upstream Dockerfile.base)
FROM node:26.13.1 AS deps

ARG APP_PATH
WORKDIR $APP_PATH
COPY ./package.json ./yarn.lock ./.yarnrc.yml ./
COPY ./patches ./patches

RUN apt-get update && apt-get install -y cmake
ENV NODE_OPTIONS="--max-old-space-size=4096"

RUN npm install -g corepack
RUN corepack enable
RUN yarn install --immutable --network-timeout 1000000 && \
  yarn cache clean

COPY . .
ARG CDN_URL
RUN yarn build

RUN yarn workspaces focus --production && \
  yarn cache clean

ENV PORT=3000

# --- runner (from upstream Dockerfile)
FROM node:26.13.1-slim AS runner

LABEL org.opencontainers.image.source="https://github.com/kafitramarna/outline"

ARG APP_PATH
WORKDIR $APP_PATH
ENV NODE_ENV=production

ENV MALLOC_ARENA_MAX=2

RUN addgroup --gid 1001 nodejs && \
    adduser --uid 1001 --ingroup nodejs nodejs && \
    mkdir -p /var/lib/outline && \
    chown -R nodejs:nodejs /var/lib/outline && \
    chown -R nodejs:nodejs $APP_PATH

COPY --from=deps --chown=nodejs:nodejs $APP_PATH/build ./build
COPY --from=deps --chown=nodejs:nodejs $APP_PATH/server ./server
COPY --from=deps --chown=nodejs:nodejs $APP_PATH/public ./public
COPY --from=deps --chown=nodejs:nodejs $APP_PATH/.sequelizerc ./.sequelizerc
COPY --from=deps --chown=nodejs:nodejs $APP_PATH/node_modules ./node_modules
COPY --from=deps --chown=nodejs:nodejs $APP_PATH/package.json ./package.json
RUN  apt-get update \
    && apt-get install -y wget \
    && rm -rf /var/lib/apt/lists/*

ENV FILE_STORAGE_LOCAL_ROOT_DIR=/var/lib/outline/data

USER nodejs

EXPOSE 3000
CMD ["node", "./build/server/index.js"]
