# IPTVBoss XC Server container

This repository publishes the headless IPTVBoss XC Server for amd64 and arm64. The current pre-release image is:

```text
ghcr.io/walrusone/iptvboss-alpha
```

Version tags such as `3.11.21` are intended for pinned deployments. The moving `alpha` tag follows the newest container published in the current channel.

## Recommended Compose setup

Download these files into one directory:

- `compose.yaml`
- `.env.example`
- `Caddyfile`

Copy the environment example and edit it:

```sh
cp .env.example .env
nano .env
```

Set `IPTVBOSS_DOMAIN` to a public hostname whose DNS points to this server. The default `COMPOSE_PROFILES=caddy` starts a bundled Caddy reverse proxy on ports `80` and `443` and keeps IPTVBoss port `8001` on host loopback.

Start the stack:

```sh
docker compose config
docker compose pull
docker compose up --detach
docker compose ps
docker compose logs --follow
```

Open `https://your-hostname/boss.php`. On a new data volume, the first visitor creates the administrator and completes bootstrap.

The bundled Caddy setup requires the hostname's DNS records to point to this server and inbound TCP ports `80` and `443` to reach it. UDP `443` is also published for HTTP/3.

## Use an existing proxy on the Docker host

If an HTTPS reverse proxy already runs directly on the Docker host, clear the profile value in `.env`:

```env
COMPOSE_PROFILES=
```

Configure that proxy to use this upstream:

```text
http://127.0.0.1:8001
```

The proxy must send `X-Forwarded-Proto: https`. Keep proxy mode enabled:

```env
IPTVBOSS_XC_BEHIND_HTTPS_PROXY=true
IPTVBOSS_HTTPS_ONLY=false
IPTVBOSS_HOST_IP=127.0.0.1
```

A proxy in another container has separate loopback networking. Attach it to a shared user-defined network and use `http://iptvboss:8001`, or publish the backend on a private host address protected by a firewall. Do not expose direct HTTP port `8001` to the Internet.

## Data and backups

IPTVBoss databases, configuration, generated XC files, caches, and logs are stored in the named `iptvboss-data` volume. Caddy uses separate `caddy-data` and `caddy-config` volumes.

Stop IPTVBoss briefly and copy its data to a dated directory before upgrades:

```sh
mkdir -p backups/2026-08-23
docker compose stop iptvboss
docker compose cp --archive iptvboss:/data/. ./backups/2026-08-23/
docker compose start iptvboss
```

Do not run `docker compose down --volumes` unless all persistent IPTVBoss and Caddy data should be deleted.

## Update or pin a version

The image repository and tag are configured independently so a future release channel can be selected without editing `compose.yaml`:

```env
IPTVBOSS_IMAGE=ghcr.io/walrusone/iptvboss-alpha
IPTVBOSS_TAG=alpha
```

For a controlled deployment, replace the tag with an exact tested version. After making a backup:

```sh
docker compose pull
docker compose up --detach
```

If a rollback is necessary, restore the matching pre-upgrade data backup before starting an older image when its database schema may differ.

## Advanced settings

The container defaults to non-root UID/GID `10001:10001`. Change `IPTVBOSS_UID` and `IPTVBOSS_GID` only when a bind-mounted host directory requires another owner.

When the published backend can be reached by untrusted peers, restrict forwarded headers to the real proxy peer addresses or CIDRs:

```env
IPTVBOSS_XC_TRUSTED_PROXIES=172.20.0.0/16,127.0.0.1/32
```

Docker address translation can make the peer appear as a bridge gateway. Use the rejected-peer address reported by IPTVBoss and include every trusted proxy hop.

Direct HTTPS is available for installations that cannot use a reverse proxy. Disable Caddy, disable proxy mode, enable HTTPS, and mount a valid PKCS#12 store at `/data/keystore.p12`:

```env
COMPOSE_PROFILES=
IPTVBOSS_XC_BEHIND_HTTPS_PROXY=false
IPTVBOSS_HTTPS_ONLY=true
IPTVBOSS_XC_KEYSTORE_PASSWORD=replace-with-keystore-password
IPTVBOSS_HOST_IP=0.0.0.0
```

The password unlocks the TLS private-key store only. It does not encrypt IPTVBoss data. Proxy mode takes precedence and does not use the keystore.

## Pull without Compose

The package is public and can be pulled without a GitHub login:

```sh
docker pull ghcr.io/walrusone/iptvboss-alpha:alpha
```
