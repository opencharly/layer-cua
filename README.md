# layer-cua

Cua Driver ([trycua/cua](https://github.com/trycua/cua)) on a live desktop, as OpenCharly
candies.

Cua Driver is the background computer-use driver: it drives a desktop's apps through the
accessibility (AT-SPI) tree and input, enumerates windows, and captures screenshots. These
candies install and wire it into a desktop VM/pod. The `cua` plugin
([`opencharly/plugin-cua`](https://github.com/opencharly/plugin-cua)) then drives it from a
charly `cua:` plan step.

## Candies

| Candy | What it does |
|---|---|
| **`cua-driver`** | Installs the pinned Cua Driver release into `/opt/cua-driver` (binary + aux files together — the driver resolves aux files relative to its real path) and symlinks `/usr/local/bin/cua-driver`, plus the runtime libraries (`libxi`, `libxkbcommon`, AT-SPI, ffmpeg/wf-recorder/ydotool). |
| **`cua-session`** | Runs the driver as a systemd **user** unit gated on `graphical-session.target` (`cua-driver serve`), plus the optional MCP endpoint `:3000` (supergateway). The driver needs `WAYLAND_DISPLAY`/`XDG_RUNTIME_DIR`/AT-SPI, which only exist inside a live session. |
| **`cua-computer-server`** | The Cua computer-server HTTP/MCP surface on `:8000` with the embedded driver — the guest service a Cua Fleet pool probes for readiness. |
| **`cua-guest`** | The guest fixings: the uinput udev rule (isolated-input route) and, only where netplan exists, the DHCP netplan a containerDisk guest needs. |
| **`cua-hyprland-plugin`** | **Opt-in, ABI-pinned** Hyprland plugin for isolated background input. Built for the exact Hyprland version the guest runs; deliberately *not* a dependency of the driver (it must be rebuilt on every Hyprland upgrade). |

## Why the pieces are split

- The driver is a **user-session daemon**, not a system service: the session wiring is its
  own candy (`cua-session`) so a box can install the binary without starting it.
- The **Hyprland plugin has no stable ABI**. Pulling it in automatically would ship a
  plugin that fails to load after any Hyprland upgrade, so it is opt-in and fail-closed
  (`hyprctl -j cua:status` must report `abi.match:true`).
- The **computer-server** is separate because it is the Cua Fleet *readiness* contract, not
  a requirement of local driver use.

## Composing

```yaml
my-desktop:
    pod:  # or vm:
        image: omarchy-something
        add_candy:
            - '@github.com/opencharly/layer-cua/candy/cua-driver:v…'
            - '@github.com/opencharly/layer-cua/candy/cua-session:v…'
            - '@github.com/opencharly/layer-cua/candy/cua-computer-server:v…'
            - '@github.com/opencharly/layer-cua/candy/cua-guest:v…'
            # opt-in, ABI-pinned:
            - '@github.com/opencharly/layer-cua/candy/cua-hyprland-plugin:v…'
```

The check beds in `opencharly/distro-omarchy` (`check-cua-*`) compose these onto the
Omarchy VM and assert the driver contract on a real desktop.

## Verified against

The pinned driver release and the guest contract were established empirically against the
Cua Omarchy Fleet image (`public.ecr.aws/k5j5w0x5/cua-omarchy-workspace`,
Omarchy 4.0.2 / Hyprland 0.56.2 / Driver 0.26.1) — see `opencharly/plan/cua-integration.md`.

## License

MIT — see `LICENSE`. The candies install Cua's own MIT-licensed artifacts; no Cua code is
vendored here.
