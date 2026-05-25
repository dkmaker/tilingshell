# GNOME 50 / Ubuntu 26.04 build notes

This branch (`v18-beta-gnome50`) is a fork base for running Tiling Shell on
GNOME Shell 50.x. It is built on top of upstream `v18.0`, which already
contains the core GNOME 50 work (metadata bump + Super-key modifier fix).

## What this branch adds on top of upstream v18.0

| Commit | Change | Why |
|---|---|---|
| `fix: alt-tab signal leak and destroy race` | PR #523 (toabctl) | Stability — closes a signal-handler leak and a race where the alt-tab popup is destroyed during show() when the modifier key is already released (GNOME bug 596695). Not GNOME-50-specific, but improves general robustness, especially on Wayland reload cycles. |
| `fix: disconnect signals before destroying tiling managers in disable()` | PR #560 (brenjn) | Stability — reorders disable() teardown so signals are disconnected before the objects they target are destroyed. Prevents callbacks firing on freed objects during extension reload. |
| `build: switch wayland-session script to Mutter devkit (GNOME 50+)` | local | The legacy `gnome-shell --nested --wayland` flow plus `MUTTER_DEBUG_*` env vars was removed in mutter 50. Replaced with `gnome-shell --devkit --virtual-monitor 1920x1080`, the new Mutter Development Kit invocation introduced in 49 and required in 50. |

## What was deliberately NOT included

- **PR #535 (bendavis78)** — adds `"50"` to metadata, fixes SUPER_MASK → MOD4_MASK, bumps esbuild to ^0.28. The metadata and SUPER fix are **already in upstream v18.0** (commits e7e40e7 and b44528c). The esbuild bump is not needed: v18.0 ships `esbuild ^0.27.2` paired with `esbuild-sass-plugin ^3.6.0`, which dependency-resolves cleanly. The 0.28 bump only matters if `esbuild-sass-plugin@3.7.0+` is pulled in.
- **PR #528 (freemans32)** — metadata-only entry for `"50"`. Redundant: already in v18.0.
- **PR #531, #534** — closed/superseded upstream.
- **Local `@girs/gnome-shell` pin to 49.0.3 + `@girs/*-17` overrides** — these were workarounds dating from before `@girs/gnome-shell@50.0.0` existed on npm. The `*-17` packages pin libmutter ABI 17 typings, which is **wrong for GNOME 50** (mutter 50 ships libmutter ABI 18 → `@girs/clutter-18`, `meta-18`, etc.). The current v18.0 dependency `@girs/gnome-shell ^49.1.0` resolves to `49.1.0` and builds cleanly against Mutter 50 at runtime, so no pin is required.
- **Local `version-name` downgrade to 17.5** — cosmetic; this *is* the v18 beta, so the upstream `18.0` value is correct.

## Build environment

- Ubuntu 26.04 LTS, GNOME Shell 50.1, Wayland session
- Node v24.14, npm 11.12
- Requires `libgio-2.0-dev-bin` package (for `glib-compile-resources`).
  Install with: `sudo apt install libgio-2.0-dev-bin`

## Build + install

```bash
npm install
npm run build
npm run install:extension
# log out / log in (Wayland) to reload the extension
```

## Future upgrade path

When upstream bumps `@girs/gnome-shell` to `^50.0.0`, the typing chain will
pull in proper `*-18` (libmutter 18) packages. At that point this branch
can be rebased and the type accuracy will improve. No source changes are
expected to be required since the runtime behaviour is already correct.
