# AGENTS.md

This repository contains CRD-focused migration prompts for Argo CD and Flux to Plural.

## Scope

These instructions apply to the entire repository.

## Working rules

- Keep the repository CRD-only.
- Keep `README.md` as a short entry point.
- Keep migration guidance inside the proper skill folders under `skills/`.
- Keep validation guidance inside `skills/validate-plural-crds/SKILL.md`.
- Do not invent unsupported CRD fields or missing operational identifiers.

## Reference sources

Use these when updating prompts or fixtures:

- [docs.plural.sh](https://docs.plural.sh)
- repositories under [github.com/pluralsh](https://github.com/pluralsh) organization, especially [github.com/pluralsh/console](https://github.com/pluralsh/console)

## Fixtures

- Keep samples minimal and realistic.
- Add fixtures only when they verify an important migration decision.
- Group samples by source system when it improves navigation.
- Parse YAML fixtures after changes.
