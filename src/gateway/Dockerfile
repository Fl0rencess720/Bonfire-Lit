FROM golang:1.24-alpine AS builder

ARG SERVICE_NAME=gateway
ARG SERVICE_PATH=src/gateway

WORKDIR /app

ENV GOPROXY=https://goproxy.cn,direct

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN go build -o /server -ldflags="-w -s" ${SERVICE_PATH}/cmd/main.go ${SERVICE_PATH}/cmd/wire_gen.go

FROM debian:stable-slim

ARG SERVICE_PATH=src/gateway

RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ stable main contrib non-free non-free-firmware\ndeb http://mirrors.tuna.tsinghua.edu.cn/debian-security stable-security main contrib non-free non-free-firmware" > /etc/apt/sources.list

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    tini \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /server /app/server
COPY --from=builder /app/${SERVICE_PATH}/configs /app/configs

ENTRYPOINT ["/usr/bin/tini", "--"]

EXPOSE 8000

CMD ["./server", "--config", "./configs"]