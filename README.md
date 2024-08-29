use this image as a base in your dockerfiles like this:

FROM adam21b/telegram-tdlib-base:latest AS tdlib-builder

FROM golang:1.21.1-alpine3.18 AS go-builder

ENV LANG=en_US.UTF-8
ENV TZ=UTC

RUN set -eux && \
    apk update && \
    apk upgrade && \
    apk add \
        bash \
        build-base \
        ca-certificates \
        curl \
        git \
        linux-headers \
        openssl-dev \
        zlib-dev

COPY --from=tdlib-builder /usr/local/include/td /usr/local/include/td/
COPY --from=tdlib-builder /usr/local/lib/libtd* /usr/local/lib/

WORKDIR /app

COPY src/go.mod ./
RUN go mod download
COPY src src

WORKDIR /app/src

RUN go build \
    -a \
    -trimpath \
    -ldflags "-s -w" \
    -o ../[your-executable-name] \
    "." && \
    ls -lah

