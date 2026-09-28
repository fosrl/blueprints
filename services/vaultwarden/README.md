# Vaultwarden

Vaultwarden is a lightweight, Bitwarden-compatible password manager server; this blueprint publishes it through Pangolin.

This blueprint runs a single public web container and stores its state under `services/vaultwarden/data/`.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- `DOMAIN` set automatically to `https://${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- simple single-container setup
- persistent vault data (SQLite database, attachments, keys) under `./data/`

With the default values, the public hostname becomes:

```text
vault.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init vaultwarden
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force vaultwarden
```

## Run

Start Vaultwarden:

```bash
./bin/blueprint up vaultwarden
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config vaultwarden
```

View logs:

```bash
./bin/blueprint logs vaultwarden
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/vaultwarden/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `vault.<BASE_DOMAIN>`
- `SIGNUPS_ALLOWED`: leave `true` to create your first account, then set it to `false` and run `./bin/blueprint up vaultwarden` again
- `ADMIN_TOKEN` (optional) to enable the `/admin` page; use an argon2 PHC string rather than plain text, wrapped in single quotes so Compose does not expand the `$` characters (`ADMIN_TOKEN='$argon2id$v=19$...'`)
- `VAULTWARDEN_TAG` if you need to pin or update the image version

## Caveats

- Bitwarden browser, desktop and mobile apps cannot complete Pangolin's SSO login. If clients fail to connect, set `RESOURCE_AUTH_SSO_ENABLED=false` in `services/vaultwarden/.env` (and check `GLOBAL_AUTH_*` in the root `.env`) so Vaultwarden's own login is the only gate.
- Vaultwarden requires HTTPS for the web vault; Pangolin provides it on the public hostname.
- Back up `services/vaultwarden/data/`.

## Updating Versions

To change Vaultwarden versions, edit `VAULTWARDEN_TAG` in `services/vaultwarden/.env` and then run:

```bash
./bin/blueprint pull vaultwarden
./bin/blueprint up vaultwarden
```

If you need direct access to the underlying Compose project:

```bash
./bin/blueprint cmd vaultwarden pull
./bin/blueprint cmd vaultwarden exec vaultwarden sh
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.vaultwarden.name`
- `pangolin.public-resources.vaultwarden.full-domain`
- `pangolin.public-resources.vaultwarden.protocol`
- `pangolin.public-resources.vaultwarden.targets[0].method`

## Files

- `docker-compose.yml`: the Vaultwarden application stack
- `.env.example`: blueprint defaults
