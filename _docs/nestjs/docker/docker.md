---
title: 도커를 이용한 Nestjs 배포 
category: nestjs
sub_category: docker
order: 1
id: 2
# permalink: /NestJs/docker
# description: 도커를 이용한 Nestjs 배포 가이드
# parent: NESTJS
# parentUrl: /NestJs/docker/
# subparent: Configuration
# subparentUrl: /Prometheus/config/
# priority: 1
---

```dockerfile

FROM node:16-alpine as builder

RUN npm install -g --force npm@latest yarn@latest

ENV NODE_ENV build
RUN mkdir /app && chown -R node:node /app
WORKDIR /app

COPY package.json yarn.lock nest-cli.json tsconfig.* bin/ src/ /app

RUN chown -R node:node /app

USER node
RUN yarn install --frozen-lockfile --force \
    && yarn build

# ---

FROM node:16-alpine

ENV NODE_ENV production
# ENV TNS_ADMIN /app/secrets/
# ENV LD_LIBRARY_PATH=/lib

RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add libaio libnsl libc6-compat curl

USER node
WORKDIR /app
ENV NODE_ENV production

COPY --from=builder /app/package*.json /app/
COPY --from=builder /app/node_modules/ /app/node_modules/
COPY --from=builder /app/dist/ /app/dist/
COPY --from=builder /app/bin/ /app/bin/

ENTRYPOINT APP_VERSION=$(/app/bin/get_version_script.sh) node dist/src/main.js

```
