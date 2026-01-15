FROM node:20-slim AS base
WORKDIR /app

FROM base AS deps
RUN corepack enable
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

FROM base AS build
RUN corepack enable
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# --- build-time args (se pasan desde docker-compose build.args o Portainer) ---
ARG LIVEKIT_URL=ws://livekit:7880
ARG LIVEKIT_API_KEY=devkey
ARG LIVEKIT_API_SECRET=secret
ARG NEXT_PUBLIC_LIVEKIT_URL

# --- env disponibles DURANTE pnpm run build (Next.js las “hornea”) ---
ENV LIVEKIT_URL=${LIVEKIT_URL} \
    LIVEKIT_API_KEY=${LIVEKIT_API_KEY} \
    LIVEKIT_API_SECRET=${LIVEKIT_API_SECRET} \
    NEXT_PUBLIC_LIVEKIT_URL=${NEXT_PUBLIC_LIVEKIT_URL}

RUN pnpm run build

FROM base AS runner
ENV NODE_ENV=production

# --- runtime env (se puede sobreescribir desde docker-compose environment) ---
ENV LIVEKIT_URL=ws://livekit:7880 \
    LIVEKIT_API_KEY=devkey \
    LIVEKIT_API_SECRET=secret

COPY --from=build /app ./
EXPOSE 3000
CMD ["node", "node_modules/next/dist/bin/next", "start", "-H", "0.0.0.0", "-p", "3000"]
