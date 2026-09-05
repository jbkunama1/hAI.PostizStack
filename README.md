# hAI.PostizStack - Ressourcenschonender Postiz Docker Stack

## 🎯 Ziel

Self-hosted Social Media Automation mit Postiz, optimiert für minimale Ressourcen (~700-900MB RAM).

## 📦 Stack-Komponenten (Phase 2)

- **Postiz** (v2.11.3) - AI-gestützter Social Media Scheduler
- **PostgreSQL 15 Alpine** - Datenbank
- **Redis 7.2 Alpine** - Background-Jobs & Caching

## 🚀 Quick Start

### 1. Vorbereitung

```bash
# Sichere Passwörter generieren
openssl rand -base64 32  # Für POSTGRES_PASSWORD
openssl rand -base64 32  # Für JWT_SECRET
```

### 2. docker-compose.yml verwenden

Du kannst die beigefügte `docker-compose.yml` direkt verwenden oder den Inhalt in Portainer einfügen.

### 3. Portainer Stack deployen

**Option A: Portainer Web Editor**
1. **Stacks** → **Add stack**
2. **Name:** `postiz`
3. **Build method:** Web editor
4. Kopiere den Inhalt der `docker-compose.yml`
5. Klicke auf **Deploy the stack**

**Option B: Git Repository**
1. **Stacks** → **Add stack**
2. **Name:** `postiz`
3. **Build method:** Repository
4. **URL:** `https://github.com/jbkunama1/hAI.PostizStack`
5. **Compose path:** `docker-compose.yml`
6. **Reference:** `main`
7. Klicke auf **Deploy the stack**

### 4. Environment-Variablen anpassen

Ersetze im `docker-compose.yml`:

- `postiz.deine-domain.de` → deine echte Domain/Subdomain
- `DEIN_DB_PASSWD` → generiertes PostgreSQL-Passwort
- `Dein-Sicherer-Random-String-Hier-12345!` → generiertes JWT_SECRET

### 5. Docker Compose CLI

```bash
docker compose up -d
```

### 6. Zugriff

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
# cloudflared Service zum Stack hinzufügen
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
# Logs prüfen
docker logs postiz
docker logs postiz-postgres
docker logs postiz-redis

# Health Status prüfen
docker compose ps
```

### 502 Bad Gateway
- Warte 1-2 Minuten (erster Start dauert länger)
- Prüfe, ob alle Container healthy sind

### RAM zu hoch
`NODE_OPTIONS` im postiz-Service anpassen:
```yaml
NODE_OPTIONS: '--max-old-space-size=192'  # Statt 256
```

## 🔄 Upgrade-Pfade

### Phase 3: Temporal hinzufügen

Für komplexe Workflows mit Retry-Logic und garantierter Ausführung. Benötigt ~1-2GB zusätzlichen RAM.

### Externe PostgreSQL nutzen

Wenn du bereits PostgreSQL betreibst:
1. `postiz-postgres` Service entfernen
2. `DATABASE_URL` anpassen:
   ```
   postgresql://host.docker.internal:5432/postiz_db
   ```

## 📝 Nächste Schritte

1. Ersten Admin-User anlegen
2. Social-Accounts über OAuth verbinden (X, LinkedIn, Instagram, etc.)
3. Erste Posts planen
4. AI Content Generation testen

## 🔗 Links

- [Postiz GitHub](https://github.com/gitroomhq/postiz-app)
- [Postiz Dokumentation](https://docs.postiz.com/)
- [Postiz Docker Compose](https://github.com/gitroomhq/postiz-docker-compose)