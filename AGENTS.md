# Agent Notes

This is a [ZMK](https://zmk.dev/) user-config repo for a `reviung41` shield on a `nice_nano_v2` controller. It does not contain the ZMK firmware source; it references the upstream ZMK module via `west`.

## Build / verify

- The only verification is a full ZMK firmware build. There are no tests, linters, or formatters in this repo.
- CI uses the upstream reusable workflow: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3` (see `.github/workflows/build.yml`).
- CI runs on push, pull request, and `workflow_dispatch`.

### Local build (from this repo root)

Uses the same steps as CI, with ZMK pinned to the manifest version in `config/west.yml`:

```bash
west init -l config
west update --fetch-opt=--filter=tree:0
west zephyr-export
west build -s zmk/app -d build -b nice_nano_v2 \
  -- -DZMK_CONFIG="$PWD/config" -DZMK_EXTRA_MODULES="$PWD" -DSHIELD=reviung41
```

The upstream CI container image is `zmkfirmware/zmk-build-arm:stable`.

## Key config files

- `build.yaml` — GitHub Actions build matrix. Currently builds one target:
  - board: `nice_nano_v2`, shield: `reviung41`.
  - Supports `board`, `shield`, `snippet`, `cmake-args`, and `artifact-name` per entry.
- `config/west.yml` — Manifest pinning ZMK to tag `v0.3` (`import: app/west.yml`). The repo is registered as `self.path: config`.
- `zephyr/module.yml` — Declares this repo as a Zephyr module with `board_root: .`, so custom boards/shields can live under `boards/` at the repo root.
- `config/reviung41.keymap` — Keymap source (Devicetree).
- `config/reviung41.conf` — Kconfig overrides. Currently enables `ZMK_RGB_UNDERGLOW`, `WS2812_STRIP`, and `ZMK_POINTING`.

## Gotchas

- Do not treat `build.yaml` as a local build script; it only generates the CI matrix.
- `west init -l config` uses `config/` as the manifest repository. The workspace root is this repo root.
- The `.gitignore` ignores `.zmk/`, which is where local ZMK tooling may drop files.
- Because `zephyr/module.yml` exists, pass `-DZMK_EXTRA_MODULES="$PWD"` when building locally so custom board/shield definitions are picked up.
- Pointing/mouse keys in the keymap depend on `CONFIG_ZMK_POINTING=y` in `config/reviung41.conf`; without it, those bindings will fail to compile.
