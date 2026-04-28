# Installation Guide for PicPeak in Podman rootless mode

## Prerequisites

- Connect as a non-root user
- Docker/Podman installed and running

## Setup Steps

1. Clone the repository and change into it:
   ```bash
   cd /path/to/picpeak
   ```

2. Copy the `.env.example` file to `.env` and edit the variables as needed:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. Build and run the containers:
   ```bash
   podman compose up
   ```
   This will build the images and install the dependencies.

## Database Initialization

1. Initialize the database and create the admin user:
   ```bash
   podman compose run backend /app/init-production.sh
   ```

## Nginx Configuration

Create an Nginx configuration file for your host. The `server_name` must match the one in your `.env` file:

```nginx
server {
    listen 80;
    server_name picpeak.x;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Support WebSocket (useful for Next.js, Vite HMR, Socket.io, etc.)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```