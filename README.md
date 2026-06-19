# factorio-image

Our own build of the Factorio headless server image, published to
`ghcr.io/behindcurtain3/factorio`. We build it ourselves so we control **when**
a new Factorio version reaches our servers, instead of waiting on an upstream
image's release cadence.

## Why

The game binary comes straight from Wube (`factorio.com/get-download/...`), not
from any third party. This repo only wraps it. Building ourselves means:

- **Timing:** a new version is built the moment Wube ships it, and the `latest`
  / `stable` tags — which the fleet follows — only move when we choose to move
  them (manual workflow run). We can delay, skip, or test a version first.
- **Contents:** our own base image and, since every host is amd64, an
  amd64-only build (no ARM/box64 layer).

## Layout

- `docker/Dockerfile` — amd64-only build. Takes `VERSION` + `SHA256` build args
  and downloads the headless tarball from Wube.
- `docker/files/`, `docker/rcon/` — the entrypoint, helper scripts, and RCON
  client, inherited verbatim from
  [factoriotools/factorio-docker](https://github.com/factoriotools/factorio-docker)
  (MIT; see `LICENSE.factoriotools`). Keeping them unchanged preserves the
  env-var / `/factorio` volume contract the control plane depends on
  (`PORT`, `RCON_PORT`, `SAVE_NAME`, `LOAD_LATEST_SAVE`, `DLC_SPACE_AGE`,
  `TOKEN`, `UPDATE_MODS_ON_START`, RCON password file, …).
- `.github/workflows/build.yml` — builds + pushes images.
- `versions.json` — the published tags, read by the control plane to populate
  its version dropdown. Updated by CI on each build.

## Releasing

- **Automatic (scheduled):** new Wube versions are built and pushed
  version-tagged only (e.g. `:2.0.77`). They do not become `latest`/`stable`.
- **Manual (workflow_dispatch):** run **build** with a `version` and tick
  `move_latest` / `move_stable` to advance those pointers — this is the
  deliberate "roll the fleet forward" action.

## Attribution

Wrapper scripts and RCON client © factoriotools, MIT. This repo trims their
multi-arch Dockerfile to amd64 and adds our own publishing workflow.
