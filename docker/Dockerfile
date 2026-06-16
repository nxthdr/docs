FROM debian:bookworm-slim AS build

# Pin Hugo (extended) for reproducible builds — apk's `hugo` floats and previously
# broke the site when it bumped past a theme-incompatible release. Keep this at/above
# the hextra theme's min_version (see themes/hextra/theme.toml).
ARG HUGO_VERSION=0.163.2
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