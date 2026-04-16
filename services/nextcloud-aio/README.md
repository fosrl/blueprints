# Nextcloud AIO

Nextcloud AIO is a Pangolin blueprint for the official Nextcloud All-in-One master container.

This blueprint follows the upstream AIO compose model, but adapts it for Pangolin by pointing the public resource at the Apache container that AIO creates dynamically after setup.

## What You Get

- the official `nextcloud-aio-mastercontainer`
- Pangolin labels that target the generated `nextcloud-aio-apache` container on `NEXTCLOUD_AIO_APACHE_PORT`
- the AIO setup interface published locally on `${NEXTCLOUD_AIO_SETUP_PORT}`
- persistent AIO state in the named volume `nextcloud_aio_mastercontainer`

With the default values, the public hostname becomes:

```text
cloud-aio.example.com
```

## Important Constraints

- The initial AIO setup still happens through the local AIO interface on `https://<server-ip>:${NEXTCLOUD_AIO_SETUP_PORT}`.
- The generated Apache container must join `${PANGOLIN_DOCKER_NETWORK}` via `APACHE_ADDITIONAL_NETWORK` for Pangolin to reach it.
- The public resource points at `nextcloud-aio-apache`, not at the master container.
- Nextcloud AIO expects Docker socket access. This blueprint is not a good fit for Podman-only deployments.

## Init

Create the service env:

```bash
./bin/blueprint init nextcloud-aio
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force nextcloud-aio
```

## Run

Start the master container:

```bash
./bin/blueprint up nextcloud-aio
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config nextcloud-aio
```

View logs:

```bash
./bin/blueprint logs nextcloud-aio
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/nextcloud-aio/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `cloud-aio.<BASE_DOMAIN>`
- `NEXTCLOUD_AIO_TAG` if you need to pin or change the AIO release
- `NEXTCLOUD_AIO_SETUP_PORT` if `8080` is already in use on the host
- `NEXTCLOUD_AIO_APACHE_PORT` if you need a different internal target port
- `NEXTCLOUD_AIO_DATADIR` before first install if you want Nextcloud data stored elsewhere

## Pangolin Labels

This blueprint intentionally uses explicit target labels because AIO creates the real public web container dynamically:

- `pangolin.public-resources.nextcloudaio.name`
- `pangolin.public-resources.nextcloudaio.full-domain`
- `pangolin.public-resources.nextcloudaio.protocol`
- `pangolin.public-resources.nextcloudaio.targets[0].method`
- `pangolin.public-resources.nextcloudaio.targets[0].hostname`
- `pangolin.public-resources.nextcloudaio.targets[0].port`

## Files

- `docker-compose.yml`: the upstream-style AIO master container adapted for Pangolin
- `.env.example`: blueprint defaults for the AIO master container and Pangolin target wiring
