# trae-skills

This repository stores Trae skills used to standardize my development workflows.

## What's inside

- `obs-plugin-dev`: Conventions for developing an OBS plugin (C++/Qt6/i18n) based on obs-plugintemplate.
- `dev-habits`: My git workflow conventions (local commits, tagging, retagging, optional version bumping).

Skills are stored under `.trae/skills/<skill-name>/SKILL.md`.

## Install into another project

Copy (or symlink) the skill directories into the target project's `.trae/skills/`:

- `.trae/skills/obs-plugin-dev/`
- `.trae/skills/dev-habits/`

## Notes on secrets

Do not commit secrets (API keys, client secrets, tokens). Prefer environment variables, `.env` (ignored), and `CMakeUserPresets.json` (ignored) for per-project local overrides.

