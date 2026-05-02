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
| `sensor-stack/` | Placeholder — currently empty |
| `crontab-guru/` | Dockerfile + nginx config for offline crontab.guru mirror |
| `wireshark_wpa_psk/` | Dockerfile + static HTML tool for WPA PSK generation |
| `docker-registry/` | Dockerfile extending `registry:2` with baked-in `auth/` htpasswd |
| `homepage/config/` | gethomepage YAML configs (services, bookmarks, settings, widgets) |

## Conventions

- **One Compose file per logical group** — do not merge unrelated services into a single file.
- **Image naming**: custom images follow `<dockerhub-user>/<name>:<YYYYMMDD>` (e.g. `michbud98/wireshark_psk:20240428`).
- **Build contexts in Compose** use absolute paths rooted at `/home/pi/Docker-Projects/docker-services/` (the deployment path on the Pi). Keep this consistent when adding new services.
- **Restart policy**: always use `restart: unless-stopped`.
- **Port mapping comments**: document each port with `# host_port:container_port/protocol — purpose`.
- **Named networks**: each Compose file declares a named bridge network (e.g. `Small-services`, `Registry`, `Portainer`).
- **Environment variables**: sensitive values go in `stack.env` (Portainer stack env file), never hard-coded in Compose files.
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
