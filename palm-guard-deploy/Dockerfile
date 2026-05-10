# Multi-stage build for the palm-guard full-stack app.
# Builds the Vite frontend, then runs the Express+Socket.IO backend
# which also serves the built frontend from ../frontend/dist.
#
# Build context expected: the root of aboodhaymouni/palm-guard.
# Copy this Dockerfile into that repo root before building.

# ---- 1. Frontend build ----
FROM node:22-alpine AS frontend-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build

# ---- 2. Backend deps ----
FROM node:22-alpine AS backend-deps
WORKDIR /app/backend
COPY backend/package*.json ./
RUN npm ci --omit=dev

# ---- 3. Runtime image ----
FROM node:22-alpine AS runtime
WORKDIR /app

ENV NODE_ENV=production \
    HOST=0.0.0.0 \
    PORT=4000 \
    PG_DB_PATH=/data/palmguard.db

# Backend code + production deps
COPY --from=backend-deps /app/backend/node_modules ./backend/node_modules
COPY backend/ ./backend/

# Built frontend (server.js looks for ../frontend/dist relative to backend/)
COPY --from=frontend-build /app/frontend/dist ./frontend/dist

# Persistent data directory (mount a volume here)
RUN mkdir -p /data && chown -R node:node /data /app
VOLUME ["/data"]
USER node

EXPOSE 4000
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:4000/api/v1/health || exit 1

WORKDIR /app/backend
CMD ["node", "--no-warnings=ExperimentalWarning", "server.js"]
