# HTTPS with Dockerized Caddy


## Prerequisites
- This exercise assumes that you have already deployed this [repo](https://github.com/ek-osnb/cloud-starter) to your Virtual machines.
- Before you can start, you need to obtain a domain. See previous exercises for instructions on how to do this.
- You need to point your domain to the public IP address of your VM. This can be done by creating an `A` record in your domain's DNS settings. The `A` record should point to the public IP address of your VM.

## Exercise:

### Step 1: Create a Caddyfile
Caddy uses a configuration file called `Caddyfile` in the project root directory to define how it should serve your website and manage TLS certificates. Create a `Caddyfile` with the following content:

```caddyfile
yourdomain.com {
    encode gzip zstd

    handle {
        reverse_proxy frontend:80
    }
}
```

Replace `yourdomain.com` with your actual domain name. This configuration tells Caddy to serve your website at `yourdomain.com` and reverse proxy requests to the `frontend` service running on port 80.

### Step 2: Create a new Docker compose file for Caddy
Create a new `docker-compose.caddy.yml` file in the project root directory with the following content:

```yaml
services:
  app:
    image: ghcr.io/ek-osnb/deploy-test/app:latest
    # restart: unless-stopped
    environment:
      APP_GIT_SHA: ${RELEASE_TAG}
      SPRING_PROFILES_ACTIVE: ${SPRING_PROFILES_ACTIVE}
      SPRING_DATASOURCE_URL: ${SPRING_DATASOURCE_URL}
      SPRING_DATASOURCE_USERNAME: ${SPRING_DATASOURCE_USERNAME}
      SPRING_DATASOURCE_PASSWORD: ${SPRING_DATASOURCE_PASSWORD}
    ports:
      - "127.0.0.1:8081:8081"
    healthcheck:
      test: [ "CMD-SHELL", "wget -qO- http://localhost:8081/actuator/health >/dev/null 2>&1 || exit 1" ]
      interval: 15s
      timeout: 2s
      retries: 3
      start_period: 30s

  frontend:
    image: ghcr.io/ek-osnb/deploy-test/frontend:latest
    ports:
      - "80"
    depends_on:
      app:
        condition: service_healthy
  
  caddy:
    image: caddy:2
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      frontend:
        condition: service_started
      app:
        condition: service_healthy

volumes:
  caddy_data:
  caddy_config:
```

This Docker Compose file defines three services: `app`, `frontend`, and `caddy`. The `caddy` service uses the official Caddy image and mounts the `Caddyfile` for configuration. It also exposes ports 80 and 443 for HTTP and HTTPS traffic.

### Step 3: Push changes and deploy
Commit and push your changes to the repository.

SSH into your VM and download the newly created `docker-compose.caddy.yml` file:

```bash
wget https://raw.githubusercontent.com/YOUR_USERNAME/cloud-starter/main/docker-compose.caddy.yml -O docker-compose.caddy.yml
```
> Make sure to replace `YOUR_USERNAME` with your actual GitHub username.

Then tear down the existing deployment and start the new one with Caddy:

```bash
docker compose -f docker-compose.prod.app.yml down
```

Then start the new deployment with Caddy:

```bash
docker compose -f docker-compose.caddy.yml up -d
```
This will start the all three services: `app`, `frontend`, and `caddy`. Caddy will automatically obtain and manage TLS certificates for your domain, so your website should now be accessible over HTTPS.

### Step 4: Verify HTTPS
Open a web browser and navigate to `https://yourdomain.com` (replace with your actual domain name). You should see your website served securely with HTTPS. You can also check the TLS certificate details in the browser to confirm that it was issued by Let's Encrypt.