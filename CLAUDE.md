# CLAUDE.md - AI Assistant Guide for self-hosted

## Project Overview

This is a **Docker Compose-based self-hosted services platform** maintained by [atareao](https://github.com/atareao) (Lorenzo Carbonell). It contains 100+ containerized applications organized as independent, modular service directories. Each service can be deployed standalone behind either **Traefik** or **Caddy** as a reverse proxy.

- **License:** MIT
- **Primary language:** Spanish (documentation, commit messages, READMEs)
- **Domain template:** `*.tuservidor.es` (placeholder for user's actual domain)

## Repository Structure

```
self-hosted/
├── .github/workflows/     # GitHub Actions (issue auto-assign)
├── .gitignore             # Excludes .env, srv/*, htpasswd, filebrowser.db
├── .gitmodules            # Git submodule: dasherr/Dasherr
├── README.md              # Root project description
├── LICENSE.md             # MIT License
├── traefik/               # Traefik reverse proxy setup
├── caddy/                 # Caddy reverse proxy setup
├── portainer+traefik/     # Combined Portainer + Traefik example
├── dev/                   # WordPress development environment
└── <service>/             # 100+ individual service directories
    ├── docker-compose.yml          # Base service definition (no proxy config)
    ├── docker-compose.traefik.yml  # Traefik labels and proxy network
    ├── docker-compose.caddy.yml    # Caddy labels and proxy network
    ├── sample.env                  # Template environment variables
    ├── README.md                   # Service-specific documentation (Spanish)
    ├── Dockerfile                  # (some services) Custom image build
    └── *.conf / *.toml / *.json    # (some services) App configuration files
```

### Key Architectural Patterns

- **Separation of concerns:** Base `docker-compose.yml` files contain NO proxy routing. Proxy configuration lives in dedicated overlay files (`docker-compose.traefik.yml` or `docker-compose.caddy.yml`).
- **Compose file merging:** Services are started by combining the base file with a proxy overlay:
  ```bash
  docker-compose -f docker-compose.yml -f docker-compose.caddy.yml up -d
  # or
  docker-compose -f docker-compose.yml -f docker-compose.traefik.yml up -d
  ```
- **Networking:** All services connect to an external `proxy` network (created by the reverse proxy setup). Internal service dependencies use a separate `internal` network.
- **Data persistence:** Local bind mounts using `./data`, `./config`, `./db` patterns relative to the service directory.

## Service Categories

| Category | Examples |
|----------|----------|
| **Infrastructure** | traefik, caddy, portainer, dockge, dozzle, yacht |
| **Media** | jellyfin, navidrome, metube, immich, photoprism, kavita |
| **Productivity** | nextcloud, memos, trilium, logseq, flatnotes, silverbullet |
| **Knowledge** | bookstack, wikijs, focalboard, planka, kanboard |
| **Monitoring** | grafana-influx-telegraf, umami, uptime-kuma, plausible, openobserve |
| **Communication** | matrix, ntfy, gotify, mattermost |
| **Development** | gitea, semaphore, open-webui, codimd, vscode |
| **File Management** | filebrowser, sftpgo, webdav, ferrishare, picoshare |
| **Security** | authentik, vaultwarden, crowdsec, radicale |
| **Finance** | firefly, odoo, facturascripts |

## Configuration Conventions

### Environment Variables

Every service with configurable values uses a `sample.env` file. Common variables:

| Variable | Purpose | Example |
|----------|---------|---------|
| `FQDN` | Fully qualified domain name | `service.tuservidor.es` |
| `TZ` | Timezone | `Europe/Madrid` |
| `PUID` / `PGID` | Container user/group IDs | `1000` |
| `POSTGRES_USER` | PostgreSQL username | `postgres` |
| `POSTGRES_PASSWORD` | PostgreSQL password | (user-defined) |
| `POSTGRES_DB` | PostgreSQL database name | `app` |
| `DB_HOST` | Database hostname | `db` |
| `JWT_SECRET` | API authentication secret | (random string) |
| `ADMIN_PASSWORD` | Admin password | (user-defined) |

### Docker Compose Patterns

**Base file** (`docker-compose.yml`):
```yaml
version: '3'
services:
  app:
    image: vendor/image:tag
    container_name: service-name
    restart: unless-stopped
    volumes:
      - ./data:/data
    networks:
      - internal

networks:
  internal: {}
  proxy:
    external: true
```

**Caddy overlay** (`docker-compose.caddy.yml`):
```yaml
services:
  app:
    networks:
      - proxy
    labels:
      - caddy="${FQDN}"
      - caddy.reverse_proxy="{{upstreams PORT}}"

networks:
  proxy:
    external: true
```

**Traefik overlay** (`docker-compose.traefik.yml`):
```yaml
services:
  app:
    networks:
      - proxy
    labels:
      - traefik.enable=true
      - traefik.http.services.SERVICE.loadbalancer.server.port=PORT
      - traefik.http.routers.SERVICE.entrypoints=web
      - traefik.http.routers.SERVICE.rule=Host(`${FQDN}`)
      - traefik.http.middlewares.SERVICE-https-redirect.redirectscheme.scheme=websecure
      - traefik.http.routers.SERVICE.middlewares=SERVICE-https-redirect
      - traefik.http.routers.SERVICE-secure.entrypoints=websecure
      - traefik.http.routers.SERVICE-secure.rule=Host(`${FQDN}`)
      - traefik.http.routers.SERVICE-secure.tls=true
      - traefik.http.routers.SERVICE-secure.tls.certresolver=letsencrypt

networks:
  proxy:
    external: true
```

## Git Conventions

### Commit Messages

- **Language:** Spanish
- **Format:** Emoji prefix + description
- **Emoji prefixes:**
  - `✨` / `:sparkles:` - New feature or functionality
  - `🐛` / `:bug:` - Bug fix
  - `🎨` - Code style improvement
  - `🚀` - Release, deployment, or version update
  - `🎉` - Celebration / major update
  - `🔧` - Configuration change

**Examples:**
```
✨ Añadido soporte para nuevas variables de entorno en configuración
:bug: Soluciona error en la validación de formularios
🚀 Actualiza la imagen de Traefik a v3.1.0 en docker-compose.yml
```

### Branch Strategy

- Main branch contains the stable configurations
- Feature branches for additions and changes

## Working with This Repository

### Adding a New Service

1. Create a directory: `mkdir <service-name>`
2. Create `docker-compose.yml` with the base service definition (no proxy config)
3. Create `docker-compose.traefik.yml` with Traefik labels
4. Create `docker-compose.caddy.yml` with Caddy labels
5. Create `sample.env` with at least the `FQDN` variable (use `service.tuservidor.es` as placeholder)
6. Create `README.md` in Spanish following the existing pattern:
   - Brief description of the service with link to upstream project
   - Installation steps (clone, cp sample.env, sed FQDN, mkdir data)
   - Startup commands for both Caddy and Traefik variants
7. Add data directories to persistence as needed

### Modifying an Existing Service

- Read the service's `docker-compose.yml`, proxy overlays, and `sample.env` first
- Maintain consistency with the existing patterns across the repository
- Keep base compose files proxy-agnostic
- Update the service README if the setup process changes

### Deploying a Service

```bash
cd self-hosted/<service>
cp sample.env .env
# Edit .env to set your actual FQDN and credentials
sed -i "s/service.tuservidor.es/your.actual.domain/g" .env

# With Caddy:
docker-compose -f docker-compose.yml -f docker-compose.caddy.yml up -d

# With Traefik:
docker-compose -f docker-compose.yml -f docker-compose.traefik.yml up -d

# View logs:
docker-compose logs -f
```

## Important Files to Never Commit

Per `.gitignore`:
- `.env` files (contain secrets and actual credentials)
- `filebrowser/filebrowser.db` (runtime database)
- `srv/*` (service data)
- `htpasswd` (password files)

## Security Considerations

- All `.env` files are gitignored - never commit actual credentials
- `sample.env` files should contain only placeholder values
- Traefik configuration includes `no-new-privileges` security option
- Services use `restart: unless-stopped` for resilience
- Internal networks isolate database and dependency traffic from the proxy network
- `htpasswd` files are used for basic auth and must not be committed

## Notes for AI Assistants

- Documentation and commit messages should be written in **Spanish**
- Use the emoji commit prefix convention described above
- Always provide both Traefik and Caddy configurations when adding services
- The domain placeholder is `tuservidor.es` (not `example.com`)
- The `proxy` network is always `external: true` and must be pre-created by the reverse proxy setup
- When creating compose files, follow the existing version format (`version: '3'` or no version for newer services)
- Service container names should match the service directory name
- Prefer Alpine-based images when available
- Use `init: true` in compose files when appropriate for proper signal handling
- Keep each service self-contained within its directory
