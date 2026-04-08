# Community Guide

This repo is intended to be a shared library of Pangolin blueprints that other people can trust, understand, and run without digging through a long thread or reverse-engineering labels.

## What Makes A Good Blueprint

- It solves one clear use case.
- It is easy to initialize with `./bin/blueprint init <service>`.
- It keeps required user edits to a minimum.
- It documents the public hostname pattern and important app-specific settings.
- It uses safe defaults and avoids unnecessary host port exposure.
- It keeps internal services private unless Pangolin actually needs to reach them.

## Submission Checklist

Before opening a PR:

1. Start with `./bin/blueprint new <service>`.
2. Make sure `./bin/blueprint init <service>` creates a usable `.env`.
3. Run `./bin/blueprint auth <service>` and `./bin/blueprint config <service>` before asking anyone else to review it.
4. Use `CHANGE_ME` for manual values and `GENERATE_<IDENTIFIER>` for generated secrets.
5. Reuse the same `GENERATE_<IDENTIFIER>` token anywhere multiple keys must receive the same generated value.
6. Confirm the public container has the expected Pangolin labels.
7. Keep the setup readable. Do not make contributors decode a giant Compose file with no explanation.
8. Add a README that explains init, run, edit points, and any important caveats.

## Review Expectations

Community blueprints should be reviewed for:

- correctness of Pangolin labels
- reasonable defaults
- clear documentation
- minimal exposed surface area
- whether secrets and state are handled cleanly

## Style Guidance

- Prefer simple names and predictable paths.
- Keep comments short and useful.
- Avoid clever shell tricks in docs.
- Optimize for the person running the blueprint for the first time.

## Future Improvements

As the repo grows, useful additions would be:

- validation tooling for blueprint metadata and labels
- CI checks for Compose rendering
- a generated index of available blueprints
- screenshots or app summaries for discovery
