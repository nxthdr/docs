FROM alpine:latest AS build

RUN apk add --update hugo

WORKDIR /opt/hugo
COPY . .
RUN hugo 

FROM caddy:latest

WORKDIR /www/html
COPY --from=build /opt/hugo/public .
COPY ./docker/config/Caddyfile /etc/caddy/Caddyfile

EXPOSE 80/tcp