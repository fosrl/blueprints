# Homepage

Homepage is a self-hosted dashboard blueprint for Pangolin.

This blueprint runs a single public web container and includes starter config files under `services/homepage/config/`.

## What You Get

- public hostname derived from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`
- tracked starter config files for services, bookmarks, and widgets
- persistent config under `./config/`

With the default values, the public hostname becomes:

```text
home.example.com
```

## Init

Create the service env:

```bash
./bin/blueprint init homepage
```

If you want to rebuild the file from the example later:

```bash
./bin/blueprint init --force homepage
```

## Run

Start Homepage:

```bash
./bin/blueprint up homepage
```

Render the final Compose config without starting containers:

```bash
./bin/blueprint config homepage
```

View logs:

```bash
./bin/blueprint logs homepage
./bin/blueprint logs-base
```

Preview generated auth labels without starting containers:

```bash
./bin/blueprint auth homepage
```

## What To Edit

After `init`, review `services/homepage/.env` and adjust:

- `SERVICE_SUBDOMAIN` if you do not want `home.<BASE_DOMAIN>`
- `HOMEPAGE_TAG` if you need to pin or update the image version
- `HOMEPAGE_TITLE` if you want a different dashboard title
- optional `RESOURCE_AUTH_*` values if you want Homepage-specific auth on top of any global defaults

Then edit the files under `services/homepage/config/`:

- `settings.yaml`
- `services.yaml`
- `bookmarks.yaml`
- `widgets.yaml`
- `docker.yaml`

## Updating Versions

To change Homepage versions, edit `HOMEPAGE_TAG` in `services/homepage/.env` and then run:

```bash
./bin/blueprint pull homepage
./bin/blueprint up homepage
```

If you need direct access to the underlying Compose project:

```bash
./bin/blueprint cmd homepage pull
./bin/blueprint cmd homepage exec homepage sh
```

## Pangolin Labels

The public app container uses these required Pangolin HTTP labels:

- `pangolin.public-resources.homepage.name`
- `pangolin.public-resources.homepage.full-domain`
- `pangolin.public-resources.homepage.protocol`
- `pangolin.public-resources.homepage.targets[0].method`

## Notes

- `HOMEPAGE_ALLOWED_HOSTS` is injected automatically from `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}`.
- The starter config is intentionally minimal so contributors can adapt it without fighting a large default dashboard.

## Testing Auth And SSO

Homepage is a good first auth test target because it is a single public container with no extra app wiring.

1. Set global auth defaults in the repo root `.env`, for example:

```env
GLOBAL_AUTH_SSO_ENABLED=true
GLOBAL_AUTH_SSO_ROLE_0=Member
GLOBAL_AUTH_SSO_ROLE_1=Support
```

2. Add any Homepage-specific auth overrides in `services/homepage/.env`, for example:

```env
RESOURCE_AUTH_SSO_ROLE_0=Ops
RESOURCE_AUTH_WHITELIST_USER_0=admin@example.com
```

3. Preview the exact generated labels:

```bash
./bin/blueprint auth homepage
```

You should see labels like:

- `pangolin.public-resources.homepage.auth.sso-enabled=true`
- `pangolin.public-resources.homepage.auth.sso-roles[0]=Member`
- `pangolin.public-resources.homepage.auth.sso-roles[1]=Support`
- `pangolin.public-resources.homepage.auth.sso-roles[2]=Ops`
- `pangolin.public-resources.homepage.auth.whitelist-users[0]=admin@example.com`

4. Start the stack:

```bash
./bin/blueprint up homepage
```

5. Confirm the final rendered project if needed:

```bash
./bin/blueprint config homepage
```

## Files

- `docker-compose.yml`: the Homepage application stack
- `.env.example`: blueprint defaults
- `config/`: starter Homepage configuration
