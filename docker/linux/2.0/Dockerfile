# AWS S3 Proxy v2.0
# docker run -d -p 8080:80 -e AWS_REGION -e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -e AWS_S3_BUCKET pottava/s3-proxy

FROM --platform=${BUILDPLATFORM} \
    golang:1.16-alpine3.13 AS builder

ARG TARGETOS
ARG TARGETARCH

RUN apk --no-cache add gcc musl-dev git
WORKDIR /go/src/github.com/pottava/aws-s3-proxy
ENV APP_VERSION=v2.0.2-eri
RUN git clone https://github.com/Erisa/aws-s3-proxy --depth=1 -b "${APP_VERSION}" .
RUN go mod download
RUN go mod verify
RUN githash=$(git rev-parse --short HEAD 2>/dev/null) \
    && today=$(date +%Y-%m-%d --utc) \
    && CGO_ENABLED=0 GOOS=linux \
    GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build \
    -ldflags '-s -w -X main.ver=${APP_VERSION} -X main.commit=${githash} -X main.date=${today}' \
    -o /app

FROM alpine:3.13 AS libs
RUN apk --no-cache add ca-certificates

FROM scratch
COPY --from=libs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app /aws-s3-proxy
ENTRYPOINT ["/aws-s3-proxy"]
