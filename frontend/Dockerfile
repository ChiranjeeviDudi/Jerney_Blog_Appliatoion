# # ---- Build stage ----
# FROM node:20-alpine AS build
# WORKDIR /app
# COPY package.json package-lock.json* ./
# RUN npm ci && npm cache clean --force
# COPY . .
# RUN npm run build

# # ---- Production stage ----
# FROM nginx:1.27-alpine AS production

# # Security: remove default nginx config
# RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# # Copy custom nginx config
# COPY nginx.conf /etc/nginx/conf.d/default.conf

# # Copy built frontend from build stage
# COPY --from=build /app/dist /usr/share/nginx/html

# # Security: run nginx as non-root
# RUN chown -R nginx:nginx /usr/share/nginx/html && \
#     chown -R nginx:nginx /var/cache/nginx && \
#     chown -R nginx:nginx /var/log/nginx && \
#     touch /var/run/nginx.pid && \
#     chown -R nginx:nginx /var/run/nginx.pid

# USER nginx

# EXPOSE 80

# CMD ["nginx", "-g", "daemon off;"]

# =========================
# Build Stage
# =========================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependency files first for better Docker cache usage
COPY package*.json ./

# Install dependencies
RUN if [ -f package-lock.json ]; then \
      npm ci; \
    else \
      npm install; \
    fi && \
    npm cache clean --force

# Copy application source
COPY . .

# Build the application
RUN npm run build

# =========================
# Production Stage
# =========================
FROM nginx:1.27-alpine

# Remove default website and configuration
RUN rm -rf /usr/share/nginx/html/* \
    && rm -f /etc/nginx/conf.d/default.conf

# Copy custom Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy build artifacts
COPY --from=builder /app/dist /usr/share/nginx/html

# Set permissions
RUN chown -R nginx:nginx /usr/share/nginx/html \
    && chown -R nginx:nginx /var/cache/nginx \
    && chown -R nginx:nginx /var/log/nginx \
    && touch /var/run/nginx.pid \
    && chown nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]