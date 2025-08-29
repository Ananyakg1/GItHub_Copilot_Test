# Secure Dockerfile for Material Dashboard 2 React
# Use a specific Node.js LTS version as the base image
FROM node:16.20.2-alpine AS build

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies with known vulnerabilities fixed
RUN npm install --omit=dev --legacy-peer-deps && npm audit fix --force

# Copy the rest of the application source code
COPY src ./src
COPY jsconfig.json ./

# Build the React app
RUN npm run build

# Use a minimal, non-root base image for serving static files
FROM nginx:1.25.5-alpine

# Create a non-root user and group
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Remove default nginx website
RUN rm -rf /usr/share/nginx/html/*

# Copy built app from previous stage
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config (if needed)
# COPY nginx.conf /etc/nginx/nginx.conf

# Set permissions
RUN chown -R appuser:appgroup /usr/share/nginx/html

# Switch to non-root user
USER appuser

# Expose port 80
EXPOSE 80

# Healthcheck for nginx
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --spider -q http://localhost || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
