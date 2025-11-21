# ---------- Stage 1: Build React App ----------
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy dependency files first for better caching
COPY package*.json ./

# Install dependencies
RUN npm install --silent

# Copy the rest of the app
COPY . .

# Build the production-ready React app
RUN npm run build

# ---------- Stage 2: Run Nginx ----------
FROM nginx:alpine

# Remove default nginx page
RUN rm -rf /usr/share/nginx/html/*

# Copy the build output from builder stage
COPY --from=builder /app/build /usr/share/nginx/html

# Optional: Custom nginx config for SPA routing (optional block)
# Uncomment the next two lines if you want client-side routing to work:
# COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose app port
EXPOSE 80

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost/ || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
