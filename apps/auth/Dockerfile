FROM node:20-slim AS builder

WORKDIR /workspace

COPY package*.json ./
COPY nx.json ./
COPY tsconfig*.json ./
COPY jest.config.ts ./
COPY jest.preset.js ./
COPY eslint.config.mjs ./
COPY webpack.*.config.js ./

COPY apps/auth ./apps/auth
COPY libs/graphql ./libs/graphql
COPY libs/grpc ./libs/grpc
COPY libs/nestjs ./libs/nestjs
COPY libs/prisma ./libs/prisma

RUN npm install

RUN apt-get update && apt-get install -y protobuf-compiler

RUN cd apps/auth && npx prisma generate

RUN npx nx build auth

FROM node:20-slim AS runner

WORKDIR /app

COPY --from=builder /workspace/package.json ./
COPY --from=builder /workspace/apps/auth/package.json ./apps/auth/package.json
COPY --from=builder /workspace/apps/graphql/package.json ./apps/graphql/package.json
COPY --from=builder /workspace/apps/grpc/package.json ./apps/grpc/package.json
COPY --from=builder /workspace/apps/auth/prisma ./prisma
COPY --from=builder /workspace/apps/auth/prisma.config.ts ./prisma.config.ts
COPY --from=builder /workspace/apps/prisma/package.json ./apps/prisma/package.json
COPY --from=builder /workspace/package-lock.json ./

ENV NODE_ENV=production

RUN npm ci

COPY --from=builder /workspace/dist ./dist

COPY --from=builder /workspace/apps/auth/src/generated ./apps/auth/src/generated

COPY apps/auth/docker-entrypoint.sh /app/docker-entrypoint.sh
RUN chmod +x /app/docker-entrypoint.sh

ENTRYPOINT [ "/app/docker-entrypoint.sh" ]