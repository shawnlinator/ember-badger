# oss-scanner build for uptime-kuma.
#
# Identical to the in-repo docker/dockerfile up to the 'release' stage, but
# stops there: the in-repo Dockerfile's final stage ('upload-artifact') is a
# CI-only release step that chmods and runs
# extra/upload-github-release-asset.sh — a script that is not part of the
# repo and needs a GitHub token — so it can never succeed in an anonymous
# scanner build (a plain `docker build` targets the last stage).
ARG BASE_IMAGE=louislam/uptime-kuma:base3

############################################
# Build in Golang
# (empty stage: provides the prebuilt healthcheck binary from the upstream
# base image, mirroring the in-repo Dockerfile)
############################################
FROM louislam/uptime-kuma:builder-go3 AS build_healthcheck

############################################
# Build in Node.js
############################################
FROM louislam/uptime-kuma:base3 AS build
USER node
WORKDIR /app

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1
COPY --chown=node:node .npmrc .npmrc
COPY --chown=node:node package.json package.json
COPY --chown=node:node package-lock.json package-lock.json
RUN npm ci --omit=dev
COPY . .
# Compile TypeScript for plain node (needs root: `COPY . .` is root-owned), then
# drop the .ts sources and the loader.
USER root
RUN npm run precompile-server && \
    find server src -name '*.ts' -delete && \
    rm -rf node_modules/tsx node_modules/esbuild node_modules/@esbuild
USER node
COPY --chown=node:node --from=build_healthcheck /app/extra/healthcheck /app/extra/healthcheck
RUN mkdir ./data

############################################
# Main Image (default target)
############################################
FROM ${BASE_IMAGE}
WORKDIR /app

LABEL org.opencontainers.image.source="https://github.com/louislam/uptime-kuma"

ENV UPTIME_KUMA_IS_CONTAINER=1
ENV NODE_ENV=production

# Copy app files from build layer
COPY --chown=node:node --from=build /app /app

EXPOSE 3001
HEALTHCHECK --interval=60s --timeout=30s --start-period=180s --retries=5 CMD extra/healthcheck
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["node", "server/server.js"]
