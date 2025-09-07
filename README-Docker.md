# Part-DB Docker Setup with Caddy

This setup runs Part-DB with SQLite database and Caddy Docker Proxy for SSL termination.

## Quick Start

1. **Ensure Caddy Docker Proxy is running:**
   ```bash
   # Make sure caddy network exists
   docker network create caddy
   ```

2. **Configure environment:**
   ```bash
   # Edit .env.docker with your settings
   nano .env.docker
   
   # Generate a secure APP_SECRET:
   openssl rand -hex 32
   ```

3. **Start Part-DB:**
   ```bash
   docker-compose --env-file .env.docker up -d
   ```

4. **Access Part-DB:**
   - URL: `https://partsdb.l.supported.systems`
   - Default login: `admin` / `admin123` (change this!)

## Configuration

### Environment Variables (.env.docker)

- `DOMAIN` - Your domain for Caddy proxy
- `INITIAL_ADMIN_PW` - Initial admin password
- `APP_SECRET` - Symfony application secret (generate with `openssl rand -hex 32`)
- `DEFAULT_CURRENCY` - Default currency (USD, EUR, etc.)

### Persistent Data

Data is stored in Docker volumes:
- `partdb_data` - SQLite database and application data
- `partdb_uploads` - Uploaded files and attachments
- `partdb_media` - Generated media files

### Backup

```bash
# Backup volumes
docker run --rm -v partdb_data:/data -v $(pwd):/backup alpine tar czf /backup/partdb-data.tar.gz /data
docker run --rm -v partdb_uploads:/data -v $(pwd):/backup alpine tar czf /backup/partdb-uploads.tar.gz /data
docker run --rm -v partdb_media:/data -v $(pwd):/backup alpine tar czf /backup/partdb-media.tar.gz /data
```

### Restore

```bash
# Restore volumes
docker run --rm -v partdb_data:/data -v $(pwd):/backup alpine tar xzf /backup/partdb-data.tar.gz -C /
docker run --rm -v partdb_uploads:/data -v $(pwd):/backup alpine tar xzf /backup/partdb-uploads.tar.gz -C /
docker run --rm -v partdb_media:/data -v $(pwd):/backup alpine tar xzf /backup/partdb-media.tar.gz -C /
```

## Upgrading

```bash
# Pull latest image
docker-compose --env-file .env.docker pull

# Restart with new image
docker-compose --env-file .env.docker up -d
```

## Troubleshooting

### Check logs
```bash
docker-compose --env-file .env.docker logs -f partdb
```

### Reset admin password
```bash
docker-compose --env-file .env.docker exec partdb php bin/console app:set-password admin
```

### Access container
```bash
docker-compose --env-file .env.docker exec partdb bash
```

## Switching to MySQL/PostgreSQL

If you need better performance later:

1. Update `docker-compose.yml` to add database service
2. Change `DATABASE_URL` in environment variables
3. Migrate data or start fresh
4. Restart containers

## Security Notes

- Change default admin password immediately
- Generate a secure `APP_SECRET`
- Keep the Docker image updated
- Consider using MySQL/PostgreSQL for production with multiple users