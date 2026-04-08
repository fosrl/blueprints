# Immich

Immich is a photo and video backup blueprint for Pangolin.

This blueprint runs:

- `immich-server`
- `immich-machine-learning`
- `immich-redis`
- `immich-database`

Only `immich-server` joins the shared Pangolin network. Redis and Postgres stay private to the blueprint.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- generated database passwords on `init`
- local app state stored under `services/immich/data/`

With the default values, the public hostname becomes:

```text
immich.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init immich
```

That generates `services/immich/.env` from `.env.example` and replaces:

- `DB_PASSWORD=GENERATE_IMMICH_DB_PASSWORD`
- `POSTGRES_PASSWORD=GENERATE_IMMICH_DB_PASSWORD`

Because both keys use the same `GENERATE_IMMICH_DB_PASSWORD` token, the generated value is shared between the app and Postgres.

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force immich
```

## Run

Start Immich:

```bash
./bin/blueprint up immich
```

This also starts `newt` if it is not already running.

Render the final Compose config without starting containers:

```bash
./bin/blueprint config immich
```

View logs:

```bash
./bin/blueprint logs immich
./bin/blueprint logs-base
```

Run any raw Compose command against the Immich stack:

```bash
./bin/blueprint cmd immich images
./bin/blueprint cmd immich exec immich-server sh
```

## What To Edit

After `init`, review `services/immich/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `immich.<BASE_DOMAIN>`
- `IMMICH_SERVER_TAG` and `IMMICH_MACHINE_LEARNING_TAG` if you need to pin or update Immich components
- `REDIS_TAG` and `POSTGRES_TAG` if you intentionally need to change the supporting image versions
- `TZ`
- `UPLOAD_LOCATION` if you want storage somewhere else
- any credentials or database values you want to override

## Updating Versions

Most users should only need to change image tags in `services/immich/.env`.

Common cases:

- update Immich app components by changing `IMMICH_SERVER_TAG` and `IMMICH_MACHINE_LEARNING_TAG`
- pin supporting images by changing `REDIS_TAG` or `POSTGRES_TAG`
- swap image registries entirely by changing the corresponding `*_IMAGE` value

After changing versions:

```bash
./bin/blueprint pull immich
./bin/blueprint up immich
```

If you want to inspect the resolved image references before starting:

```bash
./bin/blueprint config immich
```

If you want direct access to the underlying Compose project, use:

```bash
./bin/blueprint cmd immich pull
./bin/blueprint cmd immich images
./bin/blueprint cmd immich ps
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.immich.name`
- `pangolin.public-resources.immich.full-domain`
- `pangolin.public-resources.immich.protocol`
- `pangolin.public-resources.immich.targets[0].method`

This blueprint intentionally omits `hostname`, `port`, and `site` so Pangolin can auto-detect the target and attach it to the discovering Newt site.

## Files

- `docker-compose.yml`: the Immich application stack
- `.env.example`: blueprint defaults and secret placeholders
