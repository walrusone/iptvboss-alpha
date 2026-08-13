# IPTVBoss Alpha XC Server container

This repository publishes the headless IPTVBoss Alpha XC server for amd64 and
arm64:

```text
ghcr.io/walrusone/iptvboss-alpha
```

The moving `alpha` tag follows the newest container published from this Alpha
release feed. Version tags such as `3.11.21` are intended for pinned
deployments.

Each time Conveyor publishes a GitHub release containing the amd64 and arm64
Linux tarballs, GitHub Actions automatically verifies those assets and
publishes the corresponding multi-platform container. Maintainers can rerun a
missed or failed publication from **Actions → Publish XC Server Container**.

## Run with Docker Compose

Download `compose.yaml` and `.env.example`, then run:

```sh
cp .env.example .env
docker compose up --detach
docker compose logs --follow iptvboss
```

Compose defaults to direct HTTP and an all-interface listener inside the
container. This is intended for local-network setup and displays a security
warning in the console and process logs. Direct HTTP must not be
Internet-facing.

To use an HTTPS reverse proxy, set `IPTVBOSS_XC_BEHIND_HTTPS_PROXY=true`. In
the proxy, select `http`, enter the Docker server's
address as the forward hostname/IP, and enter `8001` as the forward port. The
proxy must send `X-Forwarded-Proto: https`. Open the proxy's HTTPS URL with
`/boss.php` appended to create the administrator and complete bootstrap.

Direct HTTPS is also available with:

```env
IPTVBOSS_XC_BEHIND_HTTPS_PROXY=false
IPTVBOSS_HTTPS_ONLY=true
IPTVBOSS_XC_KEYSTORE_PASSWORD=replace-with-keystore-password
```

Place `keystore.jks` in `/data`. Proxy mode takes precedence and does not use
the keystore. Access mode and listener scope are independent.
`IPTVBOSS_XC_BIND_ADDRESS` accepts `all` or `loopback`; normal bridge
networking requires `all`.

Authentication limits automatically use the forwarded client address instead
of treating the proxy container as one client. Restrict proxy headers to known
peers with a comma-separated IP/CIDR allowlist:

```env
IPTVBOSS_XC_TRUSTED_PROXIES=172.17.0.6/32,127.0.0.1/32
```

Configure this allowlist whenever the published port is reachable by untrusted
peers. Cross-bridge traffic through the Docker host may appear from a gateway
address, so use the peer reported by IPTVBoss. Include every trusted proxy
network when another proxy sits in front of Nginx Proxy Manager.

All databases, configuration, XC files, caches, and logs are stored in the
named volume `iptvboss-data`.

### Use a host directory with a custom user

The container defaults to UID/GID `10001:10001`. For a bind-mounted directory,
set `IPTVBOSS_UID` and `IPTVBOSS_GID` in `.env` to the host user and group that
should own the data. This is useful for Unraid shares, for example:

```env
IPTVBOSS_UID=99
IPTVBOSS_GID=100
```

The host directory mapped to `/data` must already exist and be writable by
that UID/GID. Changing these values for an existing named volume may require
correcting the volume's ownership first. Keep the container non-root whenever
possible.

## Pin a version

For a long-running deployment, edit `.env` and replace `alpha` with an exact
published version:

```env
IPTVBOSS_TAG=3.11.21
```

Then update the service:

```sh
docker compose pull
docker compose up --detach
```

## Configure listener and published port

IPTVBoss listens on port 8001 inside the container. To publish another host
port, edit `.env`:

```env
IPTVBOSS_HOST_PORT=8080
```

`IPTVBOSS_XC_BIND_ADDRESS` controls the listener inside the container;
`IPTVBOSS_HOST_IP` controls which host interface publishes it. Common Unraid
setups with Nginx Proxy Manager on a separate bridge use:

```env
IPTVBOSS_XC_BIND_ADDRESS=all
IPTVBOSS_HOST_IP=0.0.0.0
```

A host-local proxy can set `IPTVBOSS_HOST_IP=127.0.0.1`. Two containers merely
attached to the same bridge do not share loopback. When both services share a
user-defined bridge, use `http://iptvboss:8001` and remove the published port
when practical. A restored database must still use internal XC port 8001.

## Stop, upgrade, and back up

Stop the service without deleting its data:

```sh
docker compose down
```

Back up the `iptvboss-data` volume before changing versions. Do not use
`docker compose down --volumes` unless the persistent IPTVBoss data should be
deleted.

To follow the newest Alpha after it is published:

```sh
docker compose pull
docker compose up --detach
```

## Pull without Compose

The package is public and can be pulled without a GitHub login:

```sh
docker pull ghcr.io/walrusone/iptvboss-alpha:alpha
```
