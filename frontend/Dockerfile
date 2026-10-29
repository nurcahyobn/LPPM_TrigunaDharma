# Stage 1: Build Frontend (Node.js 22 LTS)
FROM node:22-alpine AS builder

WORKDIR /app

# Cache package installation with clean npm ci
COPY package.json package-lock.json ./
RUN npm ci

# Copy all frontend source files
COPY . .

# Argument build-time untuk URL API Backend (Vite membundel ini ke JS statis)
ARG VITE_API_URL=https://api.lppm.trigunadharma.ac.id/api/v1
ENV VITE_API_URL=$VITE_API_URL

# Jalankan build Vite
RUN npm run build

# Stage 2: Production Nginx Server
FROM nginx:1.27-alpine AS runner

# Bersihkan konfigurasi default Nginx
RUN rm -rf /etc/nginx/conf.d/* /usr/share/nginx/html/*

# Pasang konfigurasi Nginx kustom (SPA fallback, caching, security headers, healthcheck)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Salin output build dari builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
