# Language profiles

The core scaffold is language-agnostic. A profile adds the few things that differ per
language: toolchain, commands, layout, where the version is mirrored, and CI.

Profiles stay out of the always-loaded context. Agents read a profile when they set up
the project and when a task raises a language-specific question.

## Activating a profile (Phase 0)

1. Copy the profile's Commands table into `AGENTS.md` and run every command.
2. Set `profiles` and `version_mirrors` in `.scaffold.json` as the profile shows.
3. Copy the profile's `ci.yml` to `.github/workflows/ci.yml`.
4. Append the profile's ignore lines to `.gitignore`.
5. Delete the profile directories you are not using.

A repository with more than one language keeps one profile per language and prefixes the
commands in `AGENTS.md`, for example `Test (api)` and `Test (web)`.

## Adding a profile

Copy `_template/PROFILE.md`, fill every section, and keep it under about 80 lines. Put
only what an agent could not infer from the code or already know about the language.
