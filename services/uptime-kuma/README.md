# Uptime Kuma

Uptime Kuma is a self-hosted monitoring and status page blueprint for Pangolin.

This blueprint runs a single public web container and stores its state under `services/uptime-kuma/data/`.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- simple single-container setup
- persistent monitor data under `./data/`

With the default values, the public hostname becomes:

```text
status.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init uptime-kuma
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force uptime-kuma
```

## Run

Start Uptime Kuma:

```bash
./bin/blueprint up uptime-kuma
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config uptime-kuma
```

View logs:

```bash
./bin/blueprint logs uptime-kuma
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/uptime-kuma/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `status.<BASE_DOMAIN>`
- `UPTIME_KUMA_TAG` if you need to pin or update the image version

## Updating Versions

To change Uptime Kuma versions, edit `UPTIME_KUMA_TAG` in `services/uptime-kuma/.env` and then run:

```bash
./bin/blueprint pull uptime-kuma
./bin/blueprint up uptime-kuma
```

If you need direct access to the underlying Compose project:

```bash
./bin/blueprint cmd uptime-kuma pull
./bin/blueprint cmd uptime-kuma exec uptime-kuma sh
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.uptimekuma.name`
- `pangolin.public-resources.uptimekuma.full-domain`
- `pangolin.public-resources.uptimekuma.protocol`
- `pangolin.public-resources.uptimekuma.targets[0].method`

## Files

- `docker-compose.yml`: the Uptime Kuma application stack
- `.env.example`: blueprint defaults

