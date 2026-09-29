# pod-selkies

The `selkies` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It provides browser-accessible
desktop streaming over HTTPS via pixelflux Wayland capture and a Traefik reverse
proxy.

## What it provides

Installs a Python streaming server (the launch wrapper, the capture-bridge
server, and a static web-UI fileserver), a Traefik reverse proxy with a
self-signed certificate for the WebCodecs secure context, and the pre-built web
UI fleet. `pixelflux` creates the nested `wayland-1` compositor and streams H.264
to the browser; Traefik terminates TLS on `:3000` and proxies the web UI plus the
`/websockets` backend. `pcmflux` carries audio.

| Property | Value |
|---|---|
| Port | `3000` (`https+insecure` — Traefik web UI + WebSocket proxy) |
| Services | `traefik` (priority 18), `selkies-fileserver` (19), `selkies` (8) |
| Requires | `layer-supervisord`, `layer-python`, `pod-pipewire` |
| Composes | `layer-ffmpeg` (pixelflux's Wayland backend links `libav*` + `libx264`) |
| Volume | `selkies-config` → `~/.config/selkies` |
| Env | `PIXELFLUX_WAYLAND=true`, `PULSE_SERVER=unix:/tmp/pulse/native`, `LANG=C.UTF-8` |
| Source | `selkies-project/selkies` @ `af1a1c2`, built from source (fork) |

HTTPS is required because the web UI uses the WebCodecs API (`VideoDecoder`),
which needs a secure context.

## How to use it

Compose the candy into a streaming desktop box (usually via the selkies metalayer
rather than directly):

```yaml
my-selkies:
  candy:
    base: cachyos
    candy:
      - '@github.com/opencharly/pod-selkies:<tag>'
```

```bash
charly box build my-selkies
charly config my-selkies
charly start my-selkies
# open https://localhost:3000
```

## Layout

- `charly.yml` — the `selkies:` candy entity (description, `require`, `candy`,
  `env`, `port`, `security`, `volume`, `distro`, `service`, `plan`) plus its
  `skill:` entity.
- `build.sh` — the pixi builder stage: pip-installs selkies, patches
  `input_handler.py` for generic keyboard-layout support, builds the web UI.
- `selkies-wrapper`, `selkies-capture-server`, `selkies-fileserver` — the
  runtime scripts.
- `pixi.toml` / `pixi.lock` — the Python environment.
- `traefik.yml`, `traefik-dynamic.yml` — the reverse-proxy config.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-selkies:selkies` — the architecture, the capture bridge,
  the pixelflux memory-management fixes, and the build pipeline.
- `/charly-selkies:selkies-core` — the shared spine every flavor composes.
- `/charly-distros:arch-builder` — the builder image for the pixelflux from-source
  compilation.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
