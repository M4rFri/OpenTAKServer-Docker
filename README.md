# OpenTAKServer-Docker

This repo contains the docker-compose.yml and config files needed to run OpenTAKServer in a docker container.
To use it, install docker on your system, clone this repo, and run `docker compose up -d` or `sudo docker compose up -d`
depending on your platform.

This repo is new and not quite complete. Full documentation will be available on https://docs.opentakserver.io when it's ready
for use. CloudTAK will also be incorporated into docker-compose.yml.

This fork has been adapted for **Portainer CE** removing Bind-Mounts and moving Confings in the Images.

## Running behind an external Traefik

### Requirements

1. The docker network of the external Traefik must exist and be set in `.env` as `TRAEFIK_NETWORK` (default: `traefik`):
   ```
   docker network create traefik
   ```
1. `OTS_FQDN` must be a real DNS name - Traefik routes by `Host()` and `HostSNI()`.
1. Besides `web`/`websecure`, the external Traefik needs three additional entrypoints:

   ```yaml
   entryPoints:
     web:
       address: ":80"
     websecure:
       address: ":443"
     takapi:            # Marti API HTTP
       address: ":8080"
     takapisecure:      # Marti API, mTLS (TCP passthrough)
       address: ":8443"
     takace:            # Certificate enrollment (TCP passthrough)
       address: ":8446"
   ```

   The names can be changed via the `TRAEFIK_*_ENTRYPOINT` variables in `.env`.

### mTLS (Marti API, port 8443)

The Marti API requires a client certificate issued by the OTS CA. Traefik does **not** terminate TLS here. Instead a TCP router with `tls.passthrough=true` forwards the raw TLS stream to nginx based on the SNI. The client certificate check (`ssl_verify_client on`) and the server certificate stay with nginx exactly as before, so Traefik never needs access to the CA.

The same applies to enrollment on port 8446, because TAK clients expect the OTS certificate there.

> If clients connect without SNI (e.g. via an IP address), change
> ``HostSNI(`${OTS_FQDN}`)`` to ``HostSNI(`*`)`` in the labels. That is safe
> here because 8443/8446 are dedicated entrypoints used by OpenTAKServer only.
