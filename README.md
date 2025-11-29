# PostgreSQL with SSL for Dokploy

This Docker Compose setup provides a PostgreSQL database with SSL/TLS encryption and automatic certificate monitoring.

## Features

- ✅ PostgreSQL 16 with SSL/TLS enabled
- ✅ Automatic certificate file monitoring
- ✅ Automatic database restart when certificates change
- ✅ Dokploy-compatible configuration

## Prerequisites

- Docker and Docker Compose installed
- SSL certificate files:
  - `certs/fullchain.pem` - Full certificate chain
  - `certs/privkey.pem` - Private key

## Setup

1. **Create certificate directory and add your certificates:**

   ```bash
   mkdir -p certs
   # Copy your fullchain.pem and privkey.pem to the certs/ directory
   cp /path/to/your/fullchain.pem certs/
   cp /path/to/your/privkey.pem certs/
   ```

2. **Set proper permissions for certificate files:**

   ```bash
   chmod 644 certs/fullchain.pem
   chmod 600 certs/privkey.pem
   ```

3. **Configure environment variables:**

   ```bash
   cp .env.example .env
   # Edit .env with your desired PostgreSQL credentials
   ```

4. **Start the services:**

   ```bash
   docker compose up -d
   ```

## Certificate Monitoring

PostgreSQL can reload SSL certificates without a full restart using `pg_reload_conf()` or by sending a SIGHUP signal. When certificate files are replaced and PostgreSQL is signaled to reload, the new certificates will take effect.

### Automatic Monitoring (Default)

The `docker-compose.yml` includes a `cert-watcher` service that monitors certificate files (`fullchain.pem` and `privkey.pem`) for changes. When either file is modified, it automatically calls `pg_reload_conf()` to reload the SSL certificates without restarting the container.

**Benefits:**
- No container restart required (zero downtime)
- No Docker socket mounting needed (more secure)
- Automatic certificate reloading
- Compatible with Dokploy and restricted environments

### Manual Reload (Alternative)

If you prefer to manually reload certificates, you can use `docker-compose.simple.yml` and run:

```bash
# Reload configuration (including SSL certificates)
docker exec postgres-ssl psql -U postgres -c "SELECT pg_reload_conf();"

# Or send SIGHUP signal
docker kill -s SIGHUP postgres-ssl
```

## SSL Configuration

PostgreSQL is configured to:
- Require SSL connections
- Use TLS 1.2 or higher
- Use the mounted certificate files for SSL/TLS

## Connecting to the Database

### With SSL (required):

```bash
psql "postgresql://user:password@localhost:5432/dbname?sslmode=require"
```

### Connection string format:

```
postgresql://POSTGRES_USER:POSTGRES_PASSWORD@localhost:POSTGRES_PORT/POSTGRES_DB?sslmode=require
```

## Dokploy Deployment

This setup is compatible with Dokploy. When deploying:

1. Ensure the certificate files are available in the `certs/` directory
2. Set environment variables in Dokploy's environment configuration
3. Deploy using the `docker-compose.yml` file

## Troubleshooting

### Check if SSL is enabled:

```bash
docker exec postgres-ssl psql -U postgres -c "SHOW ssl;"
```

### View PostgreSQL logs:

```bash
docker logs postgres-ssl
```

### View certificate watcher logs:

```bash
docker logs postgres-cert-watcher
```

### Test SSL connection:

```bash
docker exec postgres-ssl psql -U postgres -c "SELECT version();"
```

## Security Notes

- Keep your `.env` file secure and never commit it to version control
- Ensure certificate files have proper permissions (600 for private key, 644 for certificate)
- Use strong passwords for PostgreSQL
- Consider using environment variables or secrets management in production
