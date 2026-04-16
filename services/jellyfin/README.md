# Jellyfin

Jellyfin is a self-hosted media server blueprint for Pangolin using the official container image.

This blueprint keeps Jellyfin itself public while leaving all local media mounted from a path you choose in the service env.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- persistent config and cache under `services/jellyfin/data/`
- a user-editable media mount path in `services/jellyfin/.env`

With the default values, the public hostname becomes:

```text
media.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init jellyfin
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force jellyfin
```

## Run

Start Jellyfin:

```bash
./bin/blueprint up jellyfin
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config jellyfin
```

View logs:

```bash
./bin/blueprint logs jellyfin
./bin/blueprint logs-base
```

## What To Edit

After `init`, review `services/jellyfin/.env` and adjust:

- `JELLYFIN_MEDIA_PATH` if your media library is not under `./data/media`
- `SERVICE_SUBDOMAIN` if you do not want `media.<BASE_DOMAIN>`
- `JELLYFIN_UID` and `JELLYFIN_GID` if the container should run as a different local user
- `TZ`

If you need hardware transcoding, add the required `devices:` or `group_add:` entries directly in `docker-compose.yml` for your host.

## Updating Versions

To change Jellyfin versions, edit `JELLYFIN_TAG` in `services/jellyfin/.env` and then run:

```bash
./bin/blueprint pull jellyfin
./bin/blueprint up jellyfin
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.jellyfin.name`
- `pangolin.public-resources.jellyfin.full-domain`
- `pangolin.public-resources.jellyfin.protocol`
- `pangolin.public-resources.jellyfin.targets[0].method`

## Files

- `docker-compose.yml`: the Jellyfin service definition
- `.env.example`: blueprint defaults including the media path placeholder
