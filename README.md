# Hytale Server Docker

Production-ready Docker image for running Hytale dedicated servers. Built with Java 25 and optimized for Linux VPS deployment.

## Features

- Java 25 (Eclipse Temurin/Adoptium)
- Automatic server file downloads via official Hytale downloader
- Persistent data storage with Docker volumes
- UDP/QUIC protocol support
- Simple configuration

## Requirements

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB RAM minimum (6-8GB recommended)
- Hytale game license

## Quick Start

### 1. Clone and Build

```bash
git clone https://github.com/PolGs/hytale-server-docker.git
cd hytale-server-docker
docker compose build
```

**During build**, you'll be prompted to authenticate:

```
=> => # https://oauth.accounts.hytale.com/oauth2/device/verify?user_code=gxETqUPb
=> => # Or visit the following URL and enter the code:
=> => # https://oauth.accounts.hytale.com/oauth2/device/verify
=> => # Authorization code: gxETqUPb
=> => # downloading latest ("release" patchline) to "/hytale/game.zip"
=> => # [=====                                             ] 10.0% (142.5 MB / 1.4 GB)
```

**Action:** Visit the URL and enter the code to authenticate. Build continues after authentication (~1.4 GB download, 5-10 minutes).

### 2. Start Server

```bash
docker compose up -d
```

### 3. Authenticate Server

**Important:** This is a second authentication for the running server (different from build auth).

```bash
docker attach hytale-server
```

In the console, run:

```
/auth login device
```

Follow the instructions, visit the URL, enter the code, then detach with `Ctrl+P, Ctrl+Q`.

### 4. Verify Running

```bash
docker compose logs -f
```

You should see:

```
[2026/01/13 20:25:00   INFO]   [HytaleServer] Universe ready!
[2026/01/13 20:25:00   INFO]   [HytaleServer] ===============================================================================================
[2026/01/13 20:25:00   INFO]   [HytaleServer]          Hytale Server Booted! [Multiplayer, Fresh Universe] took 22sec 201ms 220us 266ns
[2026/01/13 20:25:00   INFO]   [HytaleServer] ===============================================================================================
```

## Configuration

### Environment Variables

Edit `docker-compose.yml`:

```yaml
environment:
  - JAVA_OPTS=-Xms4G -Xmx6G  # Adjust memory for your VPS
  - HYTALE_PORT=5520          # Server port
  - HYTALE_BIND=0.0.0.0       # Bind address
```

### Port Configuration

```yaml
ports:
  - "5520:5520/udp"  # Hytale uses UDP, not TCP!
```

## Data Persistence

All server data is stored in the `hytale-data` Docker volume:

- World saves (`universe/`)
- Server configs (`config.json`, `permissions.json`, etc.)
- Mods (`mods/`)
- Logs (`logs/`)

### Managing Files

```bash
# List files
docker exec hytale-server ls -la /hytale/Server

# View config
docker exec hytale-server cat /hytale/Server/config.json

# Copy file out
docker cp hytale-server:/hytale/Server/config.json ./config.json

# Copy file in
docker cp ./config.json hytale-server:/hytale/Server/config.json
docker compose restart
```

### Installing Mods

```bash
docker cp your-mod.jar hytale-server:/hytale/Server/mods/
docker compose restart
```

### Backup

```bash
# Backup entire server
docker run --rm --volumes-from hytale-server -v $(pwd):/backup ubuntu \
  tar czf /backup/hytale-backup-$(date +%Y%m%d).tar.gz -C /hytale/Server .

# Backup just worlds
docker exec hytale-server tar czf /tmp/universe.tar.gz -C /hytale/Server/universe .
docker cp hytale-server:/tmp/universe.tar.gz ./universe-backup-$(date +%Y%m%d).tar.gz
```

## VPS Deployment

### Initial Setup

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Configure firewall (UDP, not TCP!)
sudo ufw allow 5520/udp
sudo ufw enable

# Clone and start
git clone https://github.com/PolGs/hytale-server-docker.git
cd hytale-server-docker
docker compose build
docker compose up -d

# Authenticate
docker attach hytale-server
# Run: /auth login device
# Detach: Ctrl+P, Ctrl+Q
```

### Updates

```bash
docker compose pull
docker compose up -d
```

## Docker Hub

### Publish

```bash
# Tag
docker tag hytale-server:latest yourusername/hytale-server:latest
docker tag hytale-server:latest yourusername/hytale-server:1.0.0

# Push
docker login
docker push yourusername/hytale-server:latest
docker push yourusername/hytale-server:1.0.0
```

### Use Published Image

Update `docker-compose.yml`:

```yaml
services:
  hytale-server:
    image: yourusername/hytale-server:latest
    # Remove 'build: .' line
```

## Common Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# Restart
docker compose restart

# Logs
docker compose logs -f

# Console access
docker attach hytale-server

# Detach (without stopping)
Ctrl+P, Ctrl+Q

# Resource usage
docker stats hytale-server
```

## Troubleshooting

### Players Can't Connect

1. Check firewall allows UDP port 5520:
   ```bash
   sudo ufw status
   sudo ufw allow 5520/udp
   ```

2. Verify port mapping uses `/udp` not `/tcp` in docker-compose.yml

3. Ensure server authentication completed: `/auth login device`

### High Memory Usage

Hytale's default view distance (384 blocks) uses significant RAM. To reduce:

```bash
docker cp hytale-server:/hytale/Server/config.json ./config.json
# Edit view distance
docker cp ./config.json hytale-server:/hytale/Server/config.json
docker compose restart
```

Consider the `Nitrado:PerformanceSaver` plugin for dynamic view distance adjustment.

### Server Won't Start

Check logs:
```bash
docker compose logs -f
```

Common issues:
- Insufficient memory (reduce `-Xmx` in JAVA_OPTS)
- Authentication not completed
- Port already in use

## Important Notes

- **QUIC Protocol**: Hytale uses QUIC over UDP. All firewall rules and port forwarding must be UDP, not TCP.
- **Authentication**: Two separate authentications required - one during build, one for running server.
- **Server Limit**: 100 servers per Hytale license. For more, purchase additional licenses or apply for Server Provider status.
- **Protocol Version**: Client and server must match versions exactly. Update Docker image when Hytale updates.

## Resources

- [Official Hytale Server Manual](https://support.hytale.com/hc/en-us/articles/45326769420827)
- [Server Provider Authentication Guide](https://support.hytale.com/hc/en-us/articles/45326769420827)
- [Hytale Website](https://hytale.com)

## License

Hytale and all related assets are property of Hypixel Studios. This Docker image is provided as-is for running Hytale servers.

## Support

- GitHub Issues: [Report problems](https://github.com/PolGs/hytale-server-docker/issues)
- Hytale Support: https://support.hytale.com
