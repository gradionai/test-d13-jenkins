FROM golang:1.22-alpine AS builder

WORKDIR /build
COPY go.mod .
COPY main.go .
RUN go build -o app .

FROM alpine:3.19
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /build/app .
EXPOSE 8080
CMD ["./app"]
