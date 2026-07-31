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

Open `http://localhost:8001/`. With an empty data volume, IPTVBoss starts in
bootstrap mode and presents the restore/setup page.

All databases, configuration, XC files, caches, and logs are stored in the
named volume `iptvboss-data`.

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

## Change the host port

IPTVBoss listens on port 8001 inside the container. To publish another host
port, edit `.env`:

```env
IPTVBOSS_HOST_PORT=8080
```

Browse to `http://localhost:8080/`. A restored database must still have its XC
server port set to 8001, and **Block direct connections** must be disabled.

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
