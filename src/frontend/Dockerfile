# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0


FROM node:20-alpine AS deps
RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY ./src/frontend/package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
ARG DATADOG_API_KEY
ARG DATADOG_SITE
ARG DATADOG_API_HOST
RUN apk add --no-cache libc6-compat protobuf-dev protoc
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY ./pb ./pb
COPY ./src/frontend .

RUN npm run grpc:generate
RUN npm run build
RUN npm install -g @datadog/datadog-ci
RUN datadog-ci sourcemaps upload .next/static --service=frontend-web --release-version=1.12.0 --minified-path-prefix=https://on-prem-frontend.apm-sm818-sbx-hjh.web-enedis.fr/_next/static

FROM node:20-alpine AS runner
WORKDIR /app
RUN apk add --no-cache protobuf-dev protoc

ENV NODE_ENV=production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/utils/telemetry/Instrumentation.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=deps /app/node_modules ./node_modules

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

ENV PORT=8080
EXPOSE ${PORT}

ENTRYPOINT npm start
