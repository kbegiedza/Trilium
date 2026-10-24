FROM node:24.22.1-trixie-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl git \
    && rm -rf /var/lib/apt/lists/* \
    && npm install --global --allow-scripts=pnpm pnpm@12.14.0

ENV ELECTRON_SKIP_BINARY_DOWNLOAD=1
WORKDIR /workspace
COPY . .
EXPOSE 8080
