# hAI.PostizStack - Ressourcenschonender Postiz Docker Stack

## 🎯 Ziel

Self-hosted Social Media Automation mit Postiz, optimiert für minimale Ressourcen (~700-900MB RAM).

## 📦 Stack-Komponenten (Phase 2)

- **Postiz** (v2.11.3) - AI-gestutzter Social Media Scheduler
- **PostgreSQL 15 Alpine** - Datenbank
- **Redis 7.2 Alpine** - Background-Jobs & Caching

## 🚀 Quick Start

### 1. Vorbereitung

```bash
# Sichere Passw?rter generieren
openssl rand -base64 32  # F?r POSTGRES_PASSWORD
openssl rand -base64 32  # F?r JWT_SECRET
```

### 2. docker-compose.yml erstellen

```yaml
version: '3.8'

services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:v2.11.3
    container_name: postiz
    restart: always
    environment:
      # === Required Settings (ANPASSEN!)
      MAIN_URL: 'https://postiz.deine-domain.de'
      FRONTEND_URL: 'https://postiz.deine-domain.de'
      NEXT_PUBLIC_BACKEND_URL: 'https://postiz.deine-domain.de/api'
      JWT_SECRET: 'Dein-Sicherer-Random-String-Hier-12345!'
      
      # === Database & Redis
      DATABASE_URL: '******postiz-postgres:5432/postiz-db-local'
      REDIS_URL: 'redis://postiz-redis:6379'
      
      # === Access Management
      IS_GENERAL: 'true'
      DISABLE_REGISTRATION: 'false'
      RUN_CRON: 'true'
      STORAGE_PROVIDER: 'local'
      UPLOAD_DIRECTORY: '/uploads'
      NEXT_PUBLIC_UPLOAD_DIRECTORY: '/uploads'
      
      # === Resource Limits
      NODE_OPTIONS: '--max-old-space-size=256'
    volumes:
      - postiz-config:/config/
      - postiz-uploads:/uploads/
    ports:
      - "5000:5000"
    networks:
      - postiz-network
    depends_on:
      postiz-postgres:
        condition: service_healthy
      postiz-redis:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  postiz-postgres:
    image: postgres:15-alpine
    container_name: postiz-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: DEIN_DB_PASSWD
      POSTGRES_USER: postiz-user
      POSTGRES_DB: postiz-db-local
    volumes:
      - postgres-volume:/var/lib/postgresql/data
    networks:
      - postiz-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postiz-user -d postiz-db-local"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 256M
        reservations:
          memory: 128M

  postiz-redis:
    image: redis:7.2-alpine
    container_name: postiz-redis
    restart: always
    volumes:
      - postiz-redis-data:/data
    networks:
      - postiz-network
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep -q PONG"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 128M
        reservations:
          memory: 64M

volumes:
  postgres-volume:
  postiz-redis-data:
  postiz-config:
  postiz-uploads:

networks:
  postiz-network:
```

### 3. Environment-Variablen anpassen

Ersetze im `docker-compose.yml`:

- `postiz.deine-domain.de` → deine echte Domain/Subdomain
- `DEIN_DB_PASSWD` → generiertes PostgreSQL-Passwort
- `Dein-Sicherer-Random-String-Hier-12345!` → generiertes JWT_SECRET

### 4. Stack deployen

**Option A: Docker Compose CLI**
```bash
docker compose up -d
```

**Option B: Portainer**
1. Stacks → Add stack
2. Name: `postiz`
3. Build method: Web editor
4. Code einf?gen → Deploy the stack

### 5. Zugriff

- Lokal: `http://deine-ip:5000`
- Mit Domain: `https://postiz.deine-domain.de`

## 📊 Ressourcenverbrauch

| Container | RAM Limit | Realer Verbrauch |
|-----------|-----------|------------------|
| postiz | 512M | ~300-400MB |
| postgres | 256M | ~150-250MB |
| redis | 128M | ~20-50MB |
| **Gesamt** | **896M** | **~470-700MB** |

## 🔧 Reverse Proxy Setup

### Nginx Proxy Manager

1. Neue Proxy Rule erstellen
2. Domain: `postiz.deine-domain.de`
3. Forward Host/IP: Server-IP
4. Forward Port: `5000`
5. SSL aktivieren (Let's Encrypt)

### Cloudflare Tunnel

```yaml
# cloudflared Service zum Stack hinzuf?gen
cloudflared:
  image: cloudflare/cloudflared:latest
  container_name: cloudflared
  restart: always
  command: tunnel run
  environment:
    - TUNNEL_TOKEN=DEIN_CLOUDFLARE_TUNNEL_TOKEN
  networks:
    - postiz-network
  depends_on:
    - postiz
```

## ⚠️ Troubleshooting

### Container startet nicht
```bash
# Logs pr?fen
docker logs postiz
docker logs postiz-postgres
docker logs postiz-redis

# Health Status pr?fen
docker compose ps
```

### 502 Bad Gateway
- Warte 1-2 Minuten (erster Start dauert l?nger)
- Pr?fe, ob alle Container healthy sind

### RAM zu hoch
`NODE_OPTIONS` im postiz-Service anpassen:
```yaml
NODE_OPTIONS: '--max-old-space-size=192'  # Statt 256
```

## 🔄 Upgrade-Pfade

### Phase 3: Temporal hinzuf?gen

F?r komplexe Workflows mit Retry-Logic und garantierter Ausf?hrung. Ben?tigt ~1-2GB zus?tzlichen RAM.

### Externe PostgreSQL nutzen

Wenn du bereits PostgreSQL betreibst:
1. `postiz-postgres` Service entfernen
2. `DATABASE_URL` anpassen:
   ```
   ******host.docker.internal:5432/postiz_db
   ```

## 📝 N?chste Schritte

1. Ersten Admin-User anlegen
2. Social-Accounts ?ber OAuth verbinden (X, LinkedIn, Instagram, etc.)
3. Erste Posts planen
4. AI Content Generation testen

## 🔗 Links

- [Postiz GitHub](https://github.com/gitroomhq/postiz-app)
- [Postiz Dokumentation](https://docs.postiz.com/)
- [Postiz Docker Compose](https://github.com/gitroomhq/postiz-docker-compose)