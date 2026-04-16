# Grafana

Grafana is a self-hosted observability dashboard blueprint for Pangolin using the official Docker image.

This blueprint keeps the setup intentionally small: one public Grafana container with persistent local state plus the vendored Newt Grafana examples for datasource and dashboard provisioning.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- generated admin password on `init`
- persistent Grafana state under `services/grafana/data/`
- provisioned Prometheus datasource from the pinned Newt repo example
- `Newt Overview` dashboard from the pinned Newt repo example

With the default values, the public hostname becomes:

```text
grafana.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init grafana
```

That generates `services/grafana/.env` and replaces:

- `GRAFANA_ADMIN_PASSWORD=GENERATE_GRAFANA_ADMIN_PASSWORD`

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force grafana
```

## Run

Start Grafana:

```bash
./bin/blueprint up grafana
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config grafana
```

View logs:

```bash
./bin/blueprint logs grafana
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/grafana/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `grafana.<BASE_DOMAIN>`
- `GRAFANA_TAG` if you need to pin or change the image version
- `GRAFANA_ADMIN_USER` if you want a different initial admin username

Tracked provisioning files live under:

- `services/grafana/provisioning/datasources/`
- `services/grafana/provisioning/dashboards/`
- `services/grafana/dashboards/`

The vendored datasource file currently points at `http://prometheus:9090`, matching the Prometheus blueprint container name on the shared Pangolin network. If you rename the Prometheus container, update the datasource file to match.

## Updating Versions

To change Grafana versions, edit `GRAFANA_TAG` in `services/grafana/.env` and then run:

```bash
./bin/blueprint pull grafana
./bin/blueprint up grafana
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.grafana.name`
- `pangolin.public-resources.grafana.full-domain`
- `pangolin.public-resources.grafana.protocol`
- `pangolin.public-resources.grafana.targets[0].method`

## Files

- `docker-compose.yml`: the Grafana service definition
- `.env.example`: blueprint defaults and generated admin password placeholder
- `provisioning/datasources/prometheus.yaml`: vendored Newt datasource example
- `provisioning/dashboards/dashboard.yaml`: vendored Newt dashboard provider example
- `dashboards/newt-overview.json`: vendored Newt overview dashboard
