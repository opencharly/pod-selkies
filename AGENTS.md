# AGENTS.md — pod-selkies

Standalone candy repo for the `selkies` candy — the browser-accessible desktop
streaming engine (pixelflux Wayland capture + pcmflux audio, served over HTTPS by
Traefik). The candy lives in `charly.yml` at the repo root plus its build script
and runtime wrappers.

Canonical files:

- `charly.yml` — the `selkies:` candy entity (description, `require`, `candy`,
  `env`, `port`, `security`, `volume`, `distro`, `service`, `plan`) and its
  `skill:` entity.
- `build.sh` — the pixi builder stage (pip-install selkies from the pinned
  `selkies-project/selkies` commit, patch `input_handler.py`, build the web UI).
- `selkies-wrapper`, `selkies-capture-server`, `selkies-fileserver` — the runtime
  scripts copied into `~/.local/bin`.
- `pixi.toml` / `pixi.lock` — the Python environment.
- `traefik.yml`, `traefik-dynamic.yml` — the reverse-proxy config.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:selkies` — the owning skill: the architecture, the capture
  bridge, the pixelflux memory-management fixes, and the from-source build
  pipeline. Load before editing, building, deploying, or troubleshooting this
  candy.
- `/charly-selkies:selkies-core` — the shared spine every flavor composes.
- `/charly-distros:arch-builder` / `/charly-coder:build-toolchain` — the builder
  image and the pixelflux from-source compilation dependencies.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `download:` / `check:`, service declarations).
- `/charly-check:check` — the check/R10 framework and the `cdp:` / `wl:` probe
  verbs the plan uses (`charly check run <bed>`).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert the pixelflux / Traefik / wrapper
  artifacts, the self-signed cert, the staged web UI, `GET /` returning `200`
  over HTTPS, the running `selkies` / `traefik` services, the `wayland-1` socket,
  the reachable port, a non-uniform captured frame, and an active encoder.

## Modify this repo

- Edit the `selkies:` candy entity in `charly.yml`; the `skill:` entity in the
  same file is the owning skill's source — a candy change and its skill change
  land together.
- `build.sh` is the pixi builder stage: keep the pinned `selkies-project/selkies`
  commit, the `input_handler.py` patches, and the web-UI build in step with the
  staged artifacts the plan copies.
- Compose `layer-ffmpeg` (do not merely `require:` it) — pixelflux's Wayland
  backend links `libav*` + `libx264`, and `require:` only orders deps composed
  elsewhere.
- Keep the `https+insecure:3000` port scheme and the Traefik config in step.
- The `skill:` entity is the source for `/charly-selkies:selkies`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
