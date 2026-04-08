# Contributing A Blueprint

Start with the scaffold command:

```bash
./bin/blueprint new <slug>
```

That creates `services/<slug>` from the repo template and fills the common blueprint fields automatically. Keep the result small, readable, and easy for somebody else to run without extra explanation.

## Required Files

- `docker-compose.yml`
- `.env.example`
- `.gitignore`
- `README.md`

## Blueprint Contract

- `BLUEPRINT_NAME` identifies the blueprint.
- `RESOURCE_NAME` is the human-readable Pangolin resource name.
- `SERVICE_SUBDOMAIN` is the subdomain prefix only, not the full hostname.
- `SERVICE_CONTAINER_NAME` should match the public Compose service key so generated auth labels attach to the correct service.
- `BASE_DOMAIN` always comes from the repo root `.env`.
- The public FQDN must be derived as `${SERVICE_SUBDOMAIN}.${BASE_DOMAIN}` in `pangolin.public-resources.<id>.full-domain`.
- HTTP blueprints must define `pangolin.public-resources.<id>.targets[0].method`.
- Prefer omitting `hostname`, `port`, and `site` unless the auto-detected target is wrong.
- Use `CHANGE_ME` for values users must edit manually.
- Use `GENERATE_<IDENTIFIER>` for secret placeholders that `./bin/blueprint init <service>` should replace.
- Reuse the same `GENERATE_<IDENTIFIER>` token anywhere multiple keys must receive the same generated value.
- Use `GLOBAL_AUTH_*` in the root `.env` for shared auth defaults across blueprints.
- Use `RESOURCE_AUTH_*` in the blueprint `.env` for service-specific auth overrides and additions.
- Tell users to use `./bin/blueprint init --force <service>` if they need to recreate a blueprint env from an updated example.

## Compose Guidelines

- Keep databases and caches on a blueprint-private network.
- Only attach the public app container to the shared external Pangolin network.
- Prefer bind mounts under `./data/` so state stays self-contained inside the blueprint directory.
- Add a blueprint-local `.gitignore` that ignores `.env` and persisted state directories such as `data/`.
- Avoid publishing ports unless the app truly needs a host bind.

## Documentation Expectations

- Explain what the app does in one sentence.
- Show the exact `init` and `up` commands.
- Tell users what values they are most likely to edit.
- Mention any special caveats such as large storage paths, GPU requirements, or external dependencies.

## Suggested Validation Flow

Before opening a PR, run:

```bash
./bin/blueprint init <slug>
./bin/blueprint auth <slug>
./bin/blueprint config <slug>
```

That catches most blueprint mistakes before someone else has to debug them.
