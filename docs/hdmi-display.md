# HDMI/display bring-up on J773g

This is the short, code-pointer-first description of the **working Linux
sequence**. It is hardware bring-up documentation: do not transplant it into a cold
loader or change power/firmware ordering without a recovery-qualified trial.  It describes the retained-iBoot DCP path used by the run-59
kernel, not a recipe for starting DCP firmware from a cold loader.

## Working order

1. **Keep the display fabric powered.**  The loader-visible DT keeps
   `ps_disp_fe` and `ps_dptx_phy` available and describes the main
   `dcp@30ae00000`, its DART/mailbox, five memory regions, the T8122 DPTX PHY,
   and GPIOs `hdmi-pwren`, `dp2hdmi-pwren`, `hdmi-hpd`, and
   `dp2hdmi-reset`.  Linux's M4 path calls
   `apple_pmp_j713_display_prepare()` before touching the retained session.
   Code: `drivers/gpu/drm/apple/iomfb_m4.c:iomfb_m4_init()` and
   `drivers/gpu/drm/apple/pmp-display.c`; DT/source delta:
   `private evidence receipt`
   (sections for `t8132-j773g.dts`, `t8132.dtsi`, and `iomfb_m4.c`).

2. **Prepare the fixed HDMI route before DCP adoption.**  In
   `drivers/gpu/drm/apple/dcp.c:dcp_platform_probe_pipe()`, a
   `apple,t8132-dcp` whose connector is `HDMI-A` selects the main-external
   path. `dcp_m4_hdmi_prepare()` gets `dp-phy`, runs `phy_init()` then
   `phy_power_on()`, asserts both converter power GPIOs.
   The T8122 implementation is
   `drivers/phy/apple/dptx.c:dptx_phy_activate_t8122()`; it performs the
   preserved-value PHY writes.  Source:
   the `dcp.c` and `dptx.c` sections of the same integration diff above.

3. **Adopt, do not restart, iBoot's DCP RTKit session.**
   `iomfb_m4_init()` selects the J713-compatible external profile for this
   J773g main DCP, validates the main resource, maps the
   firmware/shared regions, prepares PMP, then opens the retained DCP
   endpoints.  `dcp_pin_live_session()` deliberately keeps that session
   alive; `iomfb_m4_adopt_external()` completes the M4 handoff.  The critical
   endpoint traffic is the DCP RTKit/AFK service set, including system,
   property, IOMFB, DPTX and OS-log notifications; the AP must acknowledge
   OS-log type-2 notifications before DCP can report AP QUIESCED.  Code:
   `drivers/gpu/drm/apple/iomfb_m4.c`, `dcp.c`, `dcp-afk.c`, and
   `dcp-oslog.c`; contract source: `private evidence receipt`, rule text
   around the DCP quiesce/OS-log requirement.

4. **Connect DPTX and establish hotplug.**  After adoption,
   `dcp_desktop_dptx_connect(dcp, 0)` connects port 0.  The DPTX endpoint
   activates the PHY/lanes, trains the link, and reports HPD; the fixed-HDMI
   DRM wrapper is registered by
   `drivers/gpu/drm/apple/t8132_ext_drm.c:t8132_ext_out_init()`.
   The observed working sequence reaches `HPDMonitor: // off -> on` only
   after IOMFB switches the pipeline to normal mode.  Evidence:
   `private evidence receipt`
   lines 123-150; code delta: the `dcp.c` and `t8132_ext_drm.c` sections of
   the integration diff.

5. **Mode-set, then swap.**  DRM reads modes after HPD, performs the atomic
   modeset through the fixed external connector/CRTC, allocates an IOMFB
   surface, submits it to DCP, and waits for the firmware swap/completion
   callback before retiring the old scanout.  Follow
   `drivers/gpu/drm/apple/t8132_ext_drm.c` into
   `drivers/gpu/drm/apple/iomfb_m4.c` (surface/map, swap submission and swap
   completion functions).  The evidence records a normal-mode transition, HPD on, and subsequent pipeline programming.  The exact public function names for the final swap on
   the current rebase are **unverified** because the cited notes snapshot
   contains the topic diff rather than the complete topic tree.

## Why Aura `7c5ab9a3` is not equivalent

The Linux path powers the PHY and converter, then **adopts iBoot's already
running DCP RTKit session** and follows the kernel's endpoint/quiesce/hotplug
ordering.  Aura branch `techmore/aura:j773g/boot-hdmi-20261112@7c5ab9a3d90b`
instead adds loader-side boot-HDMI work before Linux and was composed with the
run-59 DTB and U-Boot.  Its payload hangs, so it is not a known-good display
initializer.  The precise failing Aura operation is **unverified**; do not
infer that the kernel sequence can be shortened to Aura's ordering.  Build
composition and isolation source:
`private evidence receipt`
on notes branch `j773g/boot-hdmi-20261112@bec15a4d6106`; result source: the
same run's receipt (**unverified on `origin/main` at publication time**).

## Known-good boundary

The last proven HDMI kernel is Image SHA-256 `13d3d966…` behind Stage 2
`1838b1f3…` (run 59).  Treat any change before the `HPDMonitor off -> on`
transition as power/PHY/RTKit/hotplug work, and any change after it as
modeset/surface/swap work.  Source: `private evidence receipt` and the run-59 receipt
referenced there.

Public code history: [Linux HDMI topic commit 396679fe](https://github.com/techmore/linux/commit/396679fe3659) and [Aura boot-HDMI commit 7c5ab9a3](https://github.com/techmore/aura/commit/7c5ab9a3d90b).
