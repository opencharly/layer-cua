# AGENTS.md — layer-cua

Standalone candy repo for the Cua Driver (`trycua/cua`) computer-use candies.
The members live one-per-directory under `candy/` (discovered via the `discover:`
block): `cua-driver` (the binary + aux files), `cua-session` (the user-session
systemd units), `cua-computer-server` (the HTTP/MCP surface), `cua-guest` (the
uinput udev rule + netplan), and `cua-hyprland-plugin` (the opt-in ABI-pinned
Hyprland plugin). `charly.yml` at the repo root also carries the build bed and
the test-only fixture/plugin boxes. The repo declares **no `skill:` entity**; the
owning guidance is the family skill `/charly-check:cua` (the gap is tracked in
`opencharly/opencharly#291`).

Canonical files:

- `charly.yml` — the `check-cua-driver-box` bed and the `cua-driver-test` /
  `cua-hyprland-plugin-test` boxes.
- `candy/<member>/charly.yml` — one candy per member.
- `LICENSE` — MIT.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-check:cua` — the owning (family) skill. The `cua:` check/control verb,
  the `kind: cua` entity, the read-only-by-default safety gate, and the
  containerDisk VM source. Load before editing or troubleshooting the candies.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, package sections, service declarations).
  Load before editing any entity field or plan step.
- **Missing owning skill:** these candies declare no `skill:` entity of their
  own, so no page is projected from this repo. The gap is recorded against the
  named batch
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The functional evidence is the in-repo beds: `check-cua-driver-box` (the driver
  binary + aux files, the session units on `graphical-session.target`, the uinput
  udev rule) and `cua-hyprland-plugin-test` (the plugin build + the config-wiring
  step over the Omarchy-shaped fixture). The live beds that compose these onto a
  real desktop live in `opencharly/distro-omarchy` (`check-cua-*`).
- The driver is a **user-session daemon**: it needs `WAYLAND_DISPLAY`,
  `XDG_RUNTIME_DIR`, the session D-Bus and the AT-SPI bus, so a check that
  asserts it running must be scoped to a live graphical session.

## Modify this repo

- There is no `skill:` entity here to edit; a behaviour change is mirrored only
  in the member's `candy/<name>/charly.yml` and its `plan:`.
- Keep the `cua-hyprland-plugin` deliberately opt-in and ABI-pinned — it must not
  become a dependency of `cua-driver`.
- New behaviour claims belong in a member's `plan:` as an observable `check:`
  step, and in the closest family skill.
- If the missing owning skill is authored, add the `skill:` entity here and
  update this signpost and the README in the same change.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
