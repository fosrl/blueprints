# Pangolin Blueprints

Community-maintained Docker Compose blueprints for self-hosted apps behind Pangolin.

Each blueprint lives in `services/<name>/` and is designed to work with Pangolin + Newt using Docker labels. The goal is simple:

- keep shared Pangolin/Newt config in one place
- keep each app isolated in its own directory
- make first-run setup as close as possible to `init` and `up`

## Available Blueprints

- `grafana`: dashboards and observability UI
- `homepage`: self-hosted dashboard with starter config files
- `immich`: photo and video backup
- `jellyfin`: media server
- `nextcloud`: standard Nextcloud with Redis and PostgreSQL
- `nextcloud-aio`: Nextcloud All-in-One master container adapted for Pangolin
- `prometheus`: metrics collection and querying
- `uptime-kuma`: status page and monitoring

## Create A Blueprint

Scaffold a new blueprint from the repo template:

```bash
./bin/blueprint new my-service
```

You can override the defaults if needed:

```bash
./bin/blueprint new \
  --name "My Service" \
  --subdomain my-service \
  --container-name my-service \
  --port 8080 \
  my-service
```

The scaffold command creates a new directory under `services/` and pre-fills:

- `BLUEPRINT_NAME`
- `RESOURCE_NAME`
- `SERVICE_SUBDOMAIN`
- `SERVICE_CONTAINER_NAME`
- `SERVICE_PORT`

After scaffolding, the normal contributor flow is:

```bash
./bin/blueprint init my-service
./bin/blueprint auth my-service
./bin/blueprint config my-service
```

## Quick Start

1. Create the shared repo env:

```bash
cp .env.example .env
```

2. Edit `.env` and replace every `CHANGE_ME` value.

At minimum you should set:

```env
BASE_DOMAIN=yourdomain.com
PANGOLIN_ENDPOINT=https://app.pangolin.net
NEWT_ID=CHANGE_ME
NEWT_SECRET=CHANGE_ME
```

3. See what blueprints are available:

```bash
./bin/blueprint list
```

4. Initialize a blueprint:

```bash
./bin/blueprint init <service>
```

This creates `services/<service>/.env` from the example and replaces `GENERATE_<IDENTIFIER>` secrets automatically. If the same `GENERATE_<IDENTIFIER>` token appears in multiple keys, those keys receive the same generated value.

5. Review the generated blueprint env and adjust anything app-specific.

6. Start the blueprint:

```bash
./bin/blueprint up <service>
```

`up` prints the expected public URL after the stack starts successfully.

7. Inspect the rendered config if needed:

```bash
./bin/blueprint config <service>
```

## Shared Auth

You can define auth defaults once in the root `.env` and have them applied to every blueprint resource.

Use the root `.env` for shared defaults:

```env
GLOBAL_AUTH_SSO_ENABLED=true
GLOBAL_AUTH_SSO_ROLE_0=Member
GLOBAL_AUTH_SSO_ROLE_1=Support
GLOBAL_AUTH_WHITELIST_USER_0=admin@example.com
```

If a specific blueprint needs additional or different auth labels, add `RESOURCE_AUTH_*` entries to `services/<service>/.env`:

```env
RESOURCE_AUTH_SSO_ROLE_0=Support
RESOURCE_AUTH_WHITELIST_USER_0=ops@example.com
RESOURCE_AUTH_BASIC_USER=admin
RESOURCE_AUTH_BASIC_PASSWORD=GENERATE_SERVICE_BASIC_AUTH_PASSWORD
```

The wrapper generates the Pangolin Docker labels for you, including indexed labels such as:

- `pangolin.public-resources.<id>.auth.sso-roles[0]=Member`
- `pangolin.public-resources.<id>.auth.sso-users[0]=user@example.com`
- `pangolin.public-resources.<id>.auth.whitelist-users[0]=admin@example.com`

Scalar `RESOURCE_AUTH_*` values override `GLOBAL_AUTH_*` values. Indexed `RESOURCE_AUTH_*` arrays are appended after the global arrays.

To preview the generated auth labels without starting the stack:

```bash
./bin/blueprint auth <service>
```

## Common Commands

```bash
./bin/blueprint list
./bin/blueprint new <slug>
./bin/blueprint init <service>
./bin/blueprint init --force <service>
./bin/blueprint auth <service>
./bin/blueprint up-base
./bin/blueprint up <service>
./bin/blueprint cmd <service> pull
./bin/blueprint cmd <service> exec <container> sh
./bin/blueprint logs-base
./bin/blueprint logs <service>
./bin/blueprint down <service>
```

`./bin/blueprint up <service>` also starts `newt` automatically.

Starter blueprints currently include `grafana`, `homepage`, `immich`, `jellyfin`, `nextcloud`, `nextcloud-aio`, `prometheus`, and `uptime-kuma`.

## Updating Images

Most blueprints should expose image names and tags through their service `.env` file.

The normal flow is:

1. Edit the relevant image tag in `services/<service>/.env`.
2. Pull the updated image:

```bash
./bin/blueprint pull <service>
```

3. Recreate the stack:

```bash
./bin/blueprint up <service>
```

If you need something more specific, use the generic pass-through command:

```bash
./bin/blueprint cmd <service> images
./bin/blueprint cmd <service> pull
./bin/blueprint cmd <service> restart
./bin/blueprint cmd <service> exec <container> sh
```

## How It Works

The repo is split into two layers:

- the root stack runs `newt` and owns shared Pangolin settings
- each service blueprint runs as its own Compose project under `services/<name>/`

The shared `.env` stores:

- `BASE_DOMAIN`
- `PANGOLIN_ENDPOINT`
- `NEWT_ID`
- `NEWT_SECRET`
- `NEWT_METRICS_PROMETHEUS_ENABLED`
- `NEWT_ADMIN_ADDR`
- `NEWT_ADMIN_PORT`
- `PANGOLIN_DOCKER_NETWORK`
- `HOST_CONTAINER_SOCKET`
- optional `GLOBAL_AUTH_*` defaults

Each blueprint has its own `.env` for app-specific values and optional `RESOURCE_AUTH_*` overrides.

Public hostnames are derived from the blueprint subdomain and the shared base domain:

```text
${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}
```

## Repo Layout

```text
.
|-- .env.example
|-- docker-compose.yml
|-- bin/blueprint
|-- CONTRIBUTING.md
|-- COMMUNITY.md
`-- services/
    |-- _template/
    |-- grafana/
    |-- homepage/
    |-- immich/
    |-- jellyfin/
    |-- nextcloud/
    |-- nextcloud-aio/
    |-- prometheus/
    `-- uptime-kuma/
```

## Blueprint Rules

- Every blueprint lives in `services/<slug>/`.
- Every blueprint should include `docker-compose.yml`, `.env.example`, `README.md`, and a local `.gitignore`.
- The public Compose service key should match `SERVICE_CONTAINER_NAME` so generated auth labels attach to the correct service.
- Public app containers should use Pangolin labels in the `pangolin.public-resources.<id>.*` format.
- HTTP blueprints must define `name`, `protocol`, `full-domain`, and `targets[0].method`.
- Use `CHANGE_ME` for values users must edit manually.
- Use `GENERATE_<IDENTIFIER>` for secrets the init helper should replace.
- Reuse the same `GENERATE_<IDENTIFIER>` token in multiple keys when they must share the same generated value.
- Use `GLOBAL_AUTH_*` in the root `.env` for shared auth defaults.
- Use `RESOURCE_AUTH_*` in a service `.env` to override or append auth settings for one blueprint.
- Keep databases and caches on a private blueprint network.
- Store persisted service state under `./data/` when possible so it is ignored by default.
- Only the public app container should join the shared Pangolin network.

## Community

This repo is meant to be a community catalog, not a one-off deployment repo.

If you want to contribute a blueprint:

1. Run `./bin/blueprint new <your-app>`.
2. Keep the setup small and easy to understand.
3. Make sure `./bin/blueprint init <your-app>` produces a usable `.env`.
4. Run `./bin/blueprint auth <your-app>` and `./bin/blueprint config <your-app>`.
5. Document what the blueprint exposes and what users need to change.

Start with [CONTRIBUTING.md](/opt/blueprints/CONTRIBUTING.md) and [COMMUNITY.md](/opt/blueprints/COMMUNITY.md).

## Why The Wrapper Exists

Plain `docker compose` gets clumsy when users need to remember:

- a root env
- a blueprint env
- the Newt stack
- the service stack

`./bin/blueprint` smooths that out by:

- initializing service env files
- generating placeholder secrets
- merging root and service env values for interpolation
- starting the shared Newt stack separately from each blueprint stack
- exposing a generic `cmd` escape hatch for raw Compose operations
