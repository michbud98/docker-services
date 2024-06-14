# Docker registry

## Generating password for Docker registry

```sh
mkdir -p auth
docker run -rm --entrypoint htpasswd httpd:2 -Bbn user password > auth/htpasswd
```