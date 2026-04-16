# Prometheus

Prometheus is a self-hosted metrics and alerting blueprint for Pangolin using the official container image.

This blueprint includes a tracked starter config that scrapes Prometheus itself and the shared root `newt` container, so the stack comes up with a useful default before you add real targets.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- persistent Prometheus data under `services/prometheus/data/`
- tracked starter config under `services/prometheus/config/prometheus.yml`
- default scraping for `prometheus:9090` and `newt:2112`

With the default values, the public hostname becomes:

```text
prom.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init prometheus
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force prometheus
```

## Run

Start Prometheus:

```bash
./bin/blueprint up prometheus
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config prometheus
```

View logs:

```bash
./bin/blueprint logs prometheus
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/prometheus/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `prom.<BASE_DOMAIN>`
- `PROMETHEUS_TAG` if you need to pin or change the image version
- `PROMETHEUS_RETENTION_TIME` if you want a different local retention window

Then edit `services/prometheus/config/prometheus.yml` to add your real scrape targets.

The starter config already includes:

- `prometheus:9090`
- `newt:2112`

The `newt` target assumes the root stack keeps the default `NEWT_CONTAINER_NAME=newt`. If you change that value in the root `.env`, update the Prometheus target to match.

## Updating Versions

To change Prometheus versions, edit `PROMETHEUS_TAG` in `services/prometheus/.env` and then run:

```bash
./bin/blueprint pull prometheus
./bin/blueprint up prometheus
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.prometheus.name`
- `pangolin.public-resources.prometheus.full-domain`
- `pangolin.public-resources.prometheus.protocol`
- `pangolin.public-resources.prometheus.targets[0].method`

## Files

- `docker-compose.yml`: the Prometheus service definition
- `config/prometheus.yml`: starter scrape config
- `.env.example`: blueprint defaults
