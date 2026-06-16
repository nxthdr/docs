FROM debian:bookworm-slim AS build

# Pin Hugo (extended). apk's `hugo` floats: it reached 0.160, which removed the
# .Site.Author field still referenced by the hextra theme's RSS template, breaking
# every build. Pin to a known-good version until the theme is bumped.
ARG HUGO_VERSION=0.155.0
ARG TARGETARCH
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates \
    && curl -sSL "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-${TARGETARCH}.tar.gz" \
       | tar -xz -C /usr/local/bin hugo \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt/hugo
COPY . .
RUN hugo

FROM caddy:latest

WORKDIR /www/html
COPY --from=build /opt/hugo/public .
COPY ./docker/config/Caddyfile /etc/caddy/Caddyfile

EXPOSE 80/tcp