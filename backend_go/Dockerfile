# Stage 1: Build binaries
FROM golang:1.23-alpine AS builder

WORKDIR /src

# Cache go modules
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Compile statically linked production binaries
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/server ./cmd/server \
 && CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/migrate ./cmd/migrate \
 && CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/seed ./cmd/seed \
 && CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/seed-tahap-1 ./cmd/seed-tahap-1 \
 && CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/hapus-proposal-mahasiswa ./cmd/hapus-proposal-mahasiswa

# Stage 2: Minimal & secure production runtime
FROM alpine:3.20

# Install standard CA root certificates, timezone data, and curl for health check
RUN apk add --no-cache ca-certificates tzdata curl \
 && addgroup -g 10001 -S appgroup \
 && adduser -u 10001 -S appuser -G appgroup

WORKDIR /app

# Copy compiled binaries and database migrations
COPY --from=builder /out/ /app/
COPY migrations ./migrations

# Pre-create runtime directories and set ownership to non-root appuser
RUN mkdir -p /app/logs /app/backups \
 && chown -R appuser:appgroup /app

USER appuser:appgroup

ENV PORT=3000
EXPOSE 3000

# Migrations are automatically checked and applied upon startup, then HTTP server starts
CMD ["sh", "-c", "/app/migrate && /app/server"]
