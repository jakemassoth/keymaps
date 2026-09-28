# Repository guide

## Scope and layout

This is a ZMK firmware configuration for a 38-key TOTEM split keyboard using SEEED XIAO BLE controllers, not an application or the ZMK source tree.

- `config/totem.keymap`: active user keymap. Make personal layout and behavior changes here.
- `config/totem.conf`: user Kconfig settings, including pointing support.
- `config/boards/shields/totem/`: hardware definitions, matrix transform, per-half overlays, split-role defaults, and the fallback `totem.keymap`. Do not confuse the fallback with the active user keymap or automatically keep them in sync.
- `config/west.yml`: ZMK dependency manifest, pinned to `v0.3.0`.
- `build.yaml`: firmware matrix for `totem_left`, `totem_right`, and `settings_reset`, all targeting `seeeduino_xiao_ble`.
- `.github/workflows/build.yml`: reusable ZMK build workflow, also pinned to `v0.3.0`; runs on pushes, pull requests, and manual dispatch.
- `readme.md` and `docs/images/`: setup, flashing instructions, and layout artwork.

## Editing keymaps

- Preserve the visually aligned binding rows. The matrix has 38 positions arranged as 10, 10, 12, and 6 keys, with the final row containing the thumbs. Use `totem.dtsi` as the source of truth for position order.
- Layer indices follow child-node order inside `keymap`, not node names, labels, or `#define` names. Check every `&lt`, `&mo`, and `&tog` reference when changing layer order.
- Current active layer order is BASE (0), SYM (1), NAVI (2), MOUSE (3), TVP 1 (4), TVP 2 (5). The `NAV` and `SYM` defines currently disagree with that order. Do not replace numeric references with these defines without reconciling their meaning.
- The active symbol layer currently omits the six thumb bindings. Account for this when reviewing binding counts; do not shift existing positions or silently change thumb behavior as unrelated cleanup.
- Combo `key-positions` are zero-based matrix-transform positions, not keycodes. Recheck them when moving keys.
- `&trans` falls through to lower layers; `&none` disables the position. They are not interchangeable.
- Preserve home-row modifier timing, combo timing, and TVP shortcuts unless the requested change concerns them.
- Pointing behaviors depend on the pointing header and `CONFIG_ZMK_POINTING=y`. Keep behavior references, includes, and Kconfig settings consistent.
- Use APIs and keycodes supported by the pinned ZMK release, rather than assuming current upstream documentation applies unchanged.

## Hardware and dependency changes

- The left half is the split central. The right overlay has a column offset and reversed column GPIO ordering. Avoid changing wiring or matrix definitions for a keymap-only task.
- Keep the manifest and reusable-workflow ZMK versions aligned when upgrading. Verify the board target and shield compatibility as part of any upgrade.
- Keep downloaded ZMK/Zephyr dependencies, build directories, and generated firmware out of commits.

## Validation

- Run `git diff --check` and inspect the diff for unintended binding or position changes.
- There is no repository-local test suite, formatter, or build wrapper. The existing GitHub Actions workflow is the firmware compilation check; validate all three matrix entries for firmware/build changes.
- Local builds require a separately initialized ZMK/Zephyr west workspace and toolchain. Do not present a build command as runnable from this checkout alone.
- For behavior bugs, first reproduce on the keyboard as closely as possible to the reported usage. If hardware is unavailable, state that limitation and distinguish source inspection, successful compilation, and actual device testing.
- After relevant firmware changes, hardware checks should cover both halves, layer access and exit, tap/hold behavior, combos, and any changed mouse or Bluetooth functions.
- Do not flash hardware or clear Bluetooth settings without an explicit request. `settings_reset` is recovery firmware that erases persisted settings, not a normal keymap image.
- Report which checks actually ran and which require CI or hardware. Documentation-only changes do not need a firmware build.
