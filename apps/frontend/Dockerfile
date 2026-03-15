FROM node:20-slim

WORKDIR /app

# Copy monorepo root workspace config (without lock file — regenerate for Linux platform)
COPY package.json ./

# Copy workspace packages
COPY packages/ ./packages/

# Copy frontend app
COPY apps/frontend/ ./apps/frontend/

# Install dependencies — npm resolves native binaries for the correct platform
RUN npm install --ignore-scripts

# Rebuild native modules for this platform (fixes oxc-parser and similar)
RUN npm rebuild

WORKDIR /app/apps/frontend

EXPOSE 3000
