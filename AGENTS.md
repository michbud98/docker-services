# docker-services — Agent Instructions

A collection of self-hosted Docker services deployed on a Raspberry Pi home server (`rpi-server.home.lan`). Each service is independently deployable via its own Compose file.

## Repository layout

| Path | Purpose |
|------|---------|
| `docker-compose-dns.yml` | Technitium DNS server (port 53 / 5380) |
| `docker-compose-registry.yml` | Private Docker registry (port 5000) |
| `docker-compose-small-services.yml` | Nginx-served static apps (crontab-guru :8090, wireshark-psk :8091, ascii-generator :8092) |
| `homepage/docker-compose-homepage.yml` | gethomepage dashboard (host network, port 3001) |
| `portainer/docker-compose-portainer.yml` | Portainer CE (port 9443) |
| `sensor-stack/` | Git submodule (`michbud98/cloud-intelligent-building`) — TimescaleDB + Node-RED + Grafana IoT sensor stack; see [`sensor-stack/README.md`](sensor-stack/README.md) |
| `ASCII-Generator/` | Git submodule (`michbud98/ASCII-Generator.site`) — Django app; built by `docker-compose-small-services.yml` |
| `crontab-guru/` | Dockerfile + nginx config for offline crontab.guru mirror |
| `wireshark_wpa_psk/` | Dockerfile + static HTML tool for WPA PSK generation |
| `docker-registry/` | Dockerfile extending `registry:2` with baked-in `auth/` htpasswd |
| `homepage/config/` | gethomepage YAML configs (services, bookmarks, settings, widgets) |

## Git submodules

This repo contains two submodules. After cloning, initialise them with:
```sh
git submodule update --init --recursive
```

| Submodule | Remote repo |
|-----------|-------------|
| `ASCII-Generator/` | `git@github.com:michbud98/ASCII-Generator.site.git` |
| `sensor-stack/` | `git@github.com:michbud98/cloud-intelligent-building.git` |

Submodules have their own git history and conventions. Do not commit submodule-internal changes from this repo; `cd` into the submodule and commit there.

## Conventions

- **One Compose file per logical group** — do not merge unrelated services into a single file.
- **Image naming**: custom images follow `<dockerhub-user>/<name>:<YYYYMMDD>` (e.g. `michbud98/wireshark_psk:20240428`).
- **Build contexts in Compose** use either the absolute path `/home/pi/Docker-Projects/docker-services/<dir>` or `$HOME/Docker-Projects/docker-services/<dir>`. Both forms are in use; prefer `$HOME/...` for new entries.
- **Restart policy**: use `restart: unless-stopped` (the `sensor-stack` submodule diverges and uses `restart: always`).
- **Port mapping comments**: document each port with `# host_port:container_port/protocol — purpose`.
- **Named networks**: each Compose file declares a named bridge network (e.g. `Small-services`, `Registry`, `Portainer`). Exception: `sensor-stack` uses a network named `TIG`.
- **Environment variables**: sensitive values go in `stack.env` (Portainer stack env file), never hard-coded in Compose files. Exception: `sensor-stack` uses a plain `.env` file.
- **Nginx-served services** use `nginx:1.25` as base image with a custom `config/nginx.conf` copied to `/etc/nginx/conf.d/default.conf`.

## Deployment

Services are managed through Portainer at `https://rpi-server.home.lan:9443/`. Compose files are deployed as Portainer stacks. The `stack.env` file supplies environment variables to stacks that need them (e.g. DNS server domain).

To build and push a custom image manually:
```sh
docker build -t <image-name> <service-dir>/
docker push <image-name>
```

## homepage dashboard

Configuration lives in [`homepage/config/`](homepage/config/). The dashboard runs in `network_mode: host` so it can reach LAN services directly. See [gethomepage docs](https://gethomepage.dev/) for widget/service config syntax.

## DNS server

See [`docker-compose-dns.yml`](docker-compose-dns.yml) for port options. Block lists are sourced from [firebog.net](https://firebog.net/) and [StevenBlack/hosts](https://github.com/StevenBlack/hosts). The `DNS_SERVER_DOMAIN` variable must be set in `stack.env`.

## sensor-stack

Full setup instructions are in [`sensor-stack/README.md`](sensor-stack/README.md). Key points:
- Requires a `.env` file (not `stack.env`) with `POSTGRES_PASSWORD=<value>` next to the Compose file.
- Node-RED admin password hash must be generated manually before first run (see README).
- Three Compose variants exist: `docker-compose.yml` (standard), `docker-compose-oci.yml` (OCI path variant), `docker-compose-portainer.yml` (Portainer-managed).
