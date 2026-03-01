# Multi-stage build for Auth service

# Stage 1: Builder
FROM golang:1.22-alpine AS builder

# Install build dependencies
RUN apk add --no-cache git ca-certificates

# Set working directory
WORKDIR /app

# Copy go mod and sum files
COPY go.mod go.sum ./

# Copy local dependencies (proto and shared)
# Создаём структуру папок для replace
RUN mkdir -p /proto /shared

# Копируем proto модуль (из контекста сборки)
COPY ../../proto /proto

# Копируем shared модуль (из контекста сборки)
COPY ../../shared /shared

# Download dependencies and create vendor
RUN go mod download && go mod vendor

# Copy source code
COPY . .

# Build the application using vendor
RUN CGO_ENABLED=0 GOOS=linux go build -mod=vendor -o /app/bin/auth ./cmd/auth

# Stage 2: Final image
FROM alpine:latest

# Add ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

# Create non-root user
RUN adduser -D -u 1000 appuser

# Set working directory
WORKDIR /app

# Copy binary from builder
COPY --from=builder --chown=appuser:appuser /app/bin/auth /app/auth

# Switch to non-root user
USER appuser

# Expose ports (HTTP and gRPC)
EXPOSE 8081 50051

# Command to run
CMD ["/app/auth"]