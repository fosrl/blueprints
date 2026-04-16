# Nextcloud

Nextcloud is a standard multi-container Nextcloud blueprint for Pangolin using the official Apache image with Redis and PostgreSQL.

This is the simpler, manually managed option. If you want Nextcloud's all-in-one management flow instead, use the `nextcloud-aio` blueprint.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- generated admin and database passwords on `init`
- private Redis and PostgreSQL containers on an isolated blueprint network
- persistent app and database state under `services/nextcloud/data/`

With the default values, the public hostname becomes:

```text
cloud.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init nextcloud
```

That generates `services/nextcloud/.env` and replaces:

- `NEXTCLOUD_ADMIN_PASSWORD=GENERATE_NEXTCLOUD_ADMIN_PASSWORD`
- `POSTGRES_PASSWORD=GENERATE_NEXTCLOUD_DB_PASSWORD`

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force nextcloud
```

## Run

Start Nextcloud:

```bash
./bin/blueprint up nextcloud
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config nextcloud
```

View logs:

```bash
./bin/blueprint logs nextcloud
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/nextcloud/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `cloud.<BASE_DOMAIN>`
- `NEXTCLOUD_TAG`, `POSTGRES_TAG`, or `REDIS_TAG` if you need to pin image versions
- `TZ`
- `NEXTCLOUD_ADMIN_USER` if you want a different initial admin username
- optional `RESOURCE_AUTH_*` values if you want blueprint-specific auth on top of the global defaults

## Updating Versions

Most users should only need to edit image tags in `services/nextcloud/.env`.

After changing versions:

```bash
./bin/blueprint pull nextcloud
./bin/blueprint up nextcloud
```

If you need direct access to the underlying Compose project:

```bash
./bin/blueprint cmd nextcloud pull
./bin/blueprint cmd nextcloud exec nextcloud bash
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.nextcloud.name`
- `pangolin.public-resources.nextcloud.full-domain`
- `pangolin.public-resources.nextcloud.protocol`
- `pangolin.public-resources.nextcloud.targets[0].method`

## Files

- `docker-compose.yml`: Nextcloud, Redis, and PostgreSQL services
- `.env.example`: blueprint defaults and generated-secret placeholders
