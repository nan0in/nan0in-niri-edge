# nan0in-niri-edge

Personal fork of [`StatIndet/niri-edge`](https://github.com/StatIndet/niri-edge),
which is itself a fork of the scrollable-tiling Wayland compositor
[`niri`](https://github.com/YaLTeR/niri) with native minimization support.

This repository keeps the upstream history and applies a small
"native desktop" patch used by the Clavis shell.

## Origin and references

| Item | Value |
| --- | --- |
| Original project | <https://github.com/StatIndet/niri-edge> |
| Upstream compositor | <https://github.com/YaLTeR/niri> |
| Base commit | `b2aaeb3c619610827577e4fbbb3225ce88d19baa` (2026-10-06) |
| Upstream commit title | `Merge updated upstream PR #4599, preserving minimized capture handling` |
| License | GPL-3.0, see [`LICENSE`](LICENSE) |
| Dotfiles / Clavis integration | <https://github.com/nan0in/nan0in_dotfiles> (`wm/niri-edge`) |

## Changes in this fork

### 1. Per-window `dim` rule

Adds a `dim` option to niri window rules:

```kdl
window-rule {
    // 0.0 = unchanged, 1.0 = fully black
    dim 0.08
}
```

Implementation:

- `niri-config/src/window_rule.rs`: parse `dim`.
- `src/window/mod.rs`: store and resolve the rule, clamped to `0.0..=1.0`.
- `src/layout/tile.rs`: force the clipped-surface render path while dim > 0.
- `src/render_helpers/clipped_surface.rs` and shaders: multiply only RGB by
  `1 - dim`, leaving alpha and the background blur untouched.
- `niri-config/src/lib.rs`: default value.

### 2. Native Super tap

A bare tap of either Super key (press and release within 600 ms, with no
other key, modifier, pointer, touch, tablet or gesture input) triggers the
existing unmodified `Super_L` / `Super_R` binding. This lets Clavis open its
launcher without a global keyboard listener or virtual keyboard.

Implementation:

- `src/input/mod.rs`: `ModifierTap` state machine, cancellation on pointer,
  touch, tablet, gesture, lock screen, screenshot UI, exit dialog and MRU UI.
- `src/niri.rs`: per-compositor state.
- Physical Super press/release is still forwarded to clients, so
  `wl_keyboard` modifier state stays balanced.
- VT switch and suspend clear the state.
- `src/tests/binds.rs`: release, combination, pre-held modifier, right Super,
  long-hold and client modifier-balance tests.

### 3. Packaging

`nan0in/` contains the reference patch and the systemd drop-ins used by the
private install. The drop-ins are not applied automatically; they point a
normal niri session at `~/.local/opt/niri-edge/bin/niri` and keep Clavis on the
matching IPC client.

```sh
# build
cargo build --release

# reference patch (already applied in this tree)
git apply --check nan0in/patches/0001-native-desktop.patch
```

## Notes

- The fork deliberately keeps `/usr/bin/niri` and the normal login session
  untouched. Only the private binary under `~/.local/opt/niri-edge/` is used
  when the drop-in is installed.
- The version string used by the dotfiles build is
  `26.04 (clavis-native-1 b2aaeb3)`.
- Upstream README follows below; all upstream documentation still applies.
