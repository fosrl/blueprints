# Paperless-ngx

Paperless-ngx is a document management system blueprint for Pangolin. It scans, OCRs, tags, and searches your physical documents.

This blueprint runs:

- `paperless-webserver`
- `paperless-redis`
- `paperless-database`

Only `paperless-webserver` joins the shared Pangolin network. Redis and Postgres stay private to the blueprint.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- generated admin password, database password, and Django secret key on `init`
- local app state stored under `services/paperless-ngx/data/`

With the default values, the public hostname becomes:

```text
paperless.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init paperless-ngx
```

That generates `services/paperless-ngx/.env` from `.env.example` and replaces:

- `PAPERLESS_SECRET_KEY=GENERATE_PAPERLESS_SECRET_KEY`
- `PAPERLESS_ADMIN_PASSWORD=GENERATE_PAPERLESS_ADMIN_PASSWORD`
- `PAPERLESS_DBPASS=GENERATE_PAPERLESS_DB_PASSWORD`
- `POSTGRES_PASSWORD=GENERATE_PAPERLESS_DB_PASSWORD`

Because `PAPERLESS_DBPASS` and `POSTGRES_PASSWORD` use the same `GENERATE_PAPERLESS_DB_PASSWORD` token, the generated value is shared between the app and Postgres.

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force paperless-ngx
```

## Run

Start Paperless-ngx:

```bash
./bin/blueprint up paperless-ngx
```

This also starts `newt` if it is not already running.

Render the final Compose config without starting containers:

```bash
./bin/blueprint config paperless-ngx
```

View logs:

```bash
./bin/blueprint logs paperless-ngx
./bin/blueprint logs-base
```

Run any raw Compose command against the Paperless stack:

```bash
./bin/blueprint cmd paperless-ngx images
./bin/blueprint cmd paperless-ngx exec paperless-webserver sh
```

## What To Edit

After `init`, review `services/paperless-ngx/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `paperless.<BASE_DOMAIN>`
- `PAPERLESS_URL` so it matches the full public URL
- `PAPERLESS_TIME_ZONE`
- `PAPERLESS_OCR_LANGUAGE` if you need OCR in a language other than English
- `PAPERLESS_ADMIN_USER` if you want a non-default admin username
- `PAPERLESS_TAG`, `REDIS_TAG`, `POSTGRES_TAG` if you intentionally need to change image versions

## Directories

Paperless keeps state in four directories under `services/paperless-ngx/data/`:

- `data/data`: internal application data (index, database migrations state)
- `data/media`: the processed document library
- `data/export`: export target for `document_exporter`
- `data/consume`: drop documents here for automatic ingestion

The `data/redis/` and `data/postgres/` directories hold Redis and Postgres volumes and should not be edited directly.

## Updating Versions

Most users should only need to change image tags in `services/paperless-ngx/.env`.

Common cases:

- update Paperless by changing `PAPERLESS_TAG`
- pin supporting images by changing `REDIS_TAG` or `POSTGRES_TAG`
- swap image registries entirely by changing the corresponding `*_IMAGE` value

After changing versions:

```bash
./bin/blueprint pull paperless-ngx
./bin/blueprint up paperless-ngx
```

If you want to inspect the resolved image references before starting:

```bash
./bin/blueprint config paperless-ngx
```

If you want direct access to the underlying Compose project, use:

```bash
./bin/blueprint cmd paperless-ngx pull
./bin/blueprint cmd paperless-ngx images
./bin/blueprint cmd paperless-ngx ps
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.paperless-ngx.name`
- `pangolin.public-resources.paperless-ngx.full-domain`
- `pangolin.public-resources.paperless-ngx.protocol`
- `pangolin.public-resources.paperless-ngx.targets[0].method`

This blueprint intentionally omits `hostname`, `port`, and `site` so Pangolin can auto-detect the target and attach it to the discovering Newt site.

## Files

- `docker-compose.yml`: the Paperless-ngx application stack
- `.env.example`: blueprint defaults and secret placeholders
