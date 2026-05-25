# Tiling Shell — dkmaker fork

Fork of [domferr/tilingshell](https://github.com/domferr/tilingshell) maintained for GNOME 50.1 on Ubuntu 26.04 (Wayland).

## Repo layout

- `origin` → `github.com/dkmaker/tilingshell` (fork)
- `upstream` → `github.com/domferr/tilingshell`
- Working branch: **`v18-beta-gnome50`** (based on upstream `v18.0`)
- Tag: `v18-beta-gnome50-build1` — first verified build for GNOME 50.1
- Default GitHub branch on the fork is still `main` (upstream's main) — do not commit there.

## What's on top of upstream v18.0

| Commit | Source | What |
|---|---|---|
| `cd3b644` | PR #523 (toabctl) | alt-tab: track signal IDs on MetaWindowGroup + disconnect on destroy; guard overriddenAltTab.show() against synchronous popup destruction (GNOME bug 596695) |
| `c1a52ad` | PR #560 (brenjn) | reorder disable() so `_signals.disconnect()` runs before tiling managers / indicator are destroyed — prevents callbacks on freed objects during reload |
| `bd8a0e9` | local | `wayland-session` npm script: `gnome-shell --devkit --virtual-monitor 1920x1080` (replaces removed `--nested --wayland` + `MUTTER_DEBUG_*` env) |
| `80fe193` | docs | `GNOME-50-NOTES.md` |

Upstream `v18.0` already contains the metadata bump to `"50"` and the Super-key MOD4_MASK fix — those are NOT in our delta.

## What we deliberately dropped from earlier local work

Christian's previous local checkout at `~/code-desktop/dkmaker/tilingshell` (desktop machine, never pushed) carried uncommitted workarounds. None were ported — each is obsolete:

- **`@girs/gnome-shell` pin to `49.0.3`** — dated from before `@girs/gnome-shell@50.0.0` shipped. v18.0's `^49.1.0` builds cleanly.
- **`overrides` for `@girs/*-17`** — wrong direction: GNOME 50 ships libmutter ABI **18** (`@girs/clutter-18`, `meta-18`, `mtk-18`, `shell-18`, `st-18`). The `-17` overrides pinned mutter-49-era typings.
- **`version-name` downgrade to `17.5`** — this *is* v18 beta, keep `18.0`.

## Build

```bash
sudo apt install libgio-2.0-dev-bin   # one-time, provides glib-compile-resources
npm install
npm run build                          # produces dist/ and dist_legacy/
npm run install:extension              # copies dist/ → ~/.local/share/gnome-shell/extensions/tilingshell@ferrarodomenico.com/
```

After install, log out / log in (Wayland) to reload. There is no live-reload on Wayland.

For nested dev session:
```bash
npm run dev:wayland   # builds + installs + spawns nested gnome-shell via Mutter devkit
```

## Build env baseline

- Ubuntu 26.04 LTS, GNOME Shell 50.1, Wayland
- Node v24.14, npm 11.12
- `libgio-2.0-dev-bin` 2.88.0 (for `glib-compile-resources`)

## Rebase / upgrade path

When upstream merges its GNOME 50 work into `main` and bumps `@girs/gnome-shell` to `^50.0.0`:
1. `git fetch upstream`
2. Rebase `v18-beta-gnome50` onto the new upstream tag.
3. The `*-18` typings will come in transitively; verify `dist/` still builds.
4. Re-run smoke checks: metadata, schema compile, gresource list, `disable()` ordering, MetaWindowGroup destroy hook.

## Upstream PRs to revisit

Watching but not yet included — re-evaluate before next build:

- **#523** (already cherry-picked here) — track upstream merge so we can drop our cherry-pick.
- **#560** (already cherry-picked here) — same.
- **#535** — redundant against v18.0, but if upstream merges main-only and not v18.0, our base might shift.
- **#523 / #560 / #535** all currently `OPEN / BLOCKED` upstream.

## Out-of-scope for this fork

Do not import every open PR. The branch's reason to exist is GNOME 50.1 stability on Ubuntu 26.04 — not feature accumulation. New PRs land here only with a written rationale tied to a real bug on this platform.
