<p align="center">
  <a href="https://pangolin.net">Pangolin</a> |
  <a href="https://app.pangolin.net">Pangolin Cloud</a> |
  <a href="https://docs.pangolin.net/manage/blueprints">Blueprints Docs</a>
</p>

Community repository of ready-to-use Pangolin Blueprints for applications. 

## Available Blueprints

- `grafana`: dashboards and observability UI
- `homepage`: self-hosted dashboard with starter config files
- `immich`: photo and video backup
- `jellyfin`: media server
- `nextcloud`: standard Nextcloud with Redis and PostgreSQL
- `nextcloud-aio`: Nextcloud All-in-One
- `prometheus`: metrics collection and querying
- `uptime-kuma`: status page and monitoring

## What Blueprints Are

In Pangolin, a blueprint is a declarative way to define resources and their settings without manual configuration within the dashboard. Within this repository each service has predefined labels which in turn once deployed will automatically create the corresponding resources in the Pangolin dashboard.

## Quick Start

1. Either:
  -  Create a free account at [app.pangolin.net](https://app.pangolin.net) and attach a base domain to your account.
  -  Have a [Self Hosted Pangolin](https://docs.pangolin.net/self-host/quick-install) ready to use.

2. Within the dashboard, create a site, then copy that site's configuration.

You need these three values in this repo:

```env
PANGOLIN_ENDPOINT=https://app.pangolin.net
NEWT_ID=...
NEWT_SECRET=...
```

`NEWT_ID` identifies the site connector, `NEWT_SECRET` authenticates it, and the Pangolin Cloud endpoint is `https://app.pangolin.net` or replace with your Pangolin dashboard url.

3. Clone this repository and create the shared repo env:

```bash
git clone https://github.com/fosrl/blueprints
cd blueprints && cp .env.example .env
```

4. Edit `.env` and replace every `CHANGE_ME` value.

```env
BASE_DOMAIN=yourdomain.com
PANGOLIN_ENDPOINT=https://app.pangolin.net ## change if self hosted
NEWT_ID=CHANGE_ME
NEWT_SECRET=CHANGE_ME
```

5. See what is available:

```bash
./bin/blueprint list
```

6. Initialize a blueprint:

```bash
./bin/blueprint init <service>
```

This creates `services/<service>/.env` from the example and replaces any `GENERATE_<IDENTIFIER>` placeholders automatically. If the same token appears more than once, the generated value is reused.

7. Review `services/<service>/.env` and change anything app-specific. Sanity check that `./bin/blueprint config <service>` produces configuration you expect.

8. Start it:

```bash
./bin/blueprint up <service>
```

`up` also starts `newt` automatically and prints the expected public URL when the stack comes up cleanly.

Useful follow-up commands:

```bash
./bin/blueprint config <service>
./bin/blueprint logs <service>
./bin/blueprint down <service>
```

## Shared Auth Defaults

Define shared auth once in the root `.env`:

```env
GLOBAL_AUTH_SSO_ENABLED=true
GLOBAL_AUTH_SSO_ROLE_0=Member
GLOBAL_AUTH_SSO_ROLE_1=Support
GLOBAL_AUTH_WHITELIST_USER_0=admin@example.com
```

Override or extend auth for one blueprint in `services/<service>/.env`:

```env
RESOURCE_AUTH_SSO_ROLE_0=Support
RESOURCE_AUTH_WHITELIST_USER_0=ops@example.com
RESOURCE_AUTH_BASIC_USER=admin
RESOURCE_AUTH_BASIC_PASSWORD=GENERATE_SERVICE_BASIC_AUTH_PASSWORD
```

Scalar `RESOURCE_AUTH_*` values override `GLOBAL_AUTH_*` values. Indexed `RESOURCE_AUTH_*` arrays are appended after the global arrays.

Preview the generated labels without starting the stack:

```bash
./bin/blueprint auth <service>
```

## Updating Images

Most blueprints expose image names and tags through `services/<service>/.env`.

Typical flow:

1. Edit the relevant image tag in `services/<service>/.env`.
2. Pull the updated image:

```bash
./bin/blueprint pull <service>
```

3. Recreate the stack:

```bash
./bin/blueprint up <service>
```

For raw Compose operations, use:

```bash
./bin/blueprint cmd <service> images
./bin/blueprint cmd <service> pull
./bin/blueprint cmd <service> restart
./bin/blueprint cmd <service> exec <container> sh
```

## Create A Blueprint

Scaffold a new blueprint from the template:

```bash
./bin/blueprint new my-service
```

Override the defaults if needed:

```bash
./bin/blueprint new \
  --name "My Service" \
  --subdomain my-service \
  --container-name my-service \
  --port 8080 \
  my-service
```

After scaffolding:

```bash
./bin/blueprint init my-service
./bin/blueprint auth my-service
./bin/blueprint config my-service
```

## How It Is Organized

- The root stack runs `newt` and owns the shared Pangolin connection.
- Each blueprint runs as its own Compose project under `services/<name>/`.
- The root `.env` stores shared values such as `BASE_DOMAIN`, `PANGOLIN_ENDPOINT`, `NEWT_ID`, `NEWT_SECRET`, `PANGOLIN_DOCKER_NETWORK`, and optional `GLOBAL_AUTH_*` defaults.
- Each blueprint has its own `.env` for app-specific values and optional `RESOURCE_AUTH_*` overrides.
- Public hostnames are derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`.

## Contributing

If you want to add a blueprint:

1. Run `./bin/blueprint new <your-app>`.
2. Keep the setup small and easy to understand.
3. Make sure `./bin/blueprint init <your-app>` produces a usable `.env`.
4. Run `./bin/blueprint auth <your-app>` and `./bin/blueprint config <your-app>`.
5. Document what the blueprint exposes and what users need to change.

Start with [CONTRIBUTING.md](CONTRIBUTING.md) and [COMMUNITY.md](COMMUNITY.md).

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE). Individual services may have their own upstream licenses and terms.
