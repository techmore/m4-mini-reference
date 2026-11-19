# NixOS on J773g

> **This page describes the 26.6.2 (25G83) install as of 2026-11-13.** The
> machine has since been reinstalled on the macOS 27.2 (26B123) stub. What is
> known about NixOS there is in the next section; the changes it depends on
> are **open pull requests, and nothing is merged upstream**.

## On the 27.2 stub

State on 2026-11-18, one machine, local builds, with 2026-11-19 updates
where a line says so. Background in
[Booting on the 27.2 stub](booting-27-2-stub.md).

- **It boots unattended to KDE**, through Stage 1, Stage 2, U-Boot and GRUB,
  with the root filesystem on btrfs, Ethernet up and SSH reachable about 26
  seconds after a restart.
- **Not as installed.** The image contains a prebuilt kernel made for the
  26.6.2 stub. On 27.2 that kernel resets within seconds. The system came up
  only after the kernel image on the boot partition was replaced by hand with
  a 27.2-capable build (the Stage 2 on the ESP was replaced as well; whether
  that was needed is not established). The image has to be rebuilt around
  such a kernel before a fresh install can boot by itself. [nixos-aurora #302](https://github.com/aurora-silicon/nixos-aurora/pull/302), an
  opt-in profile with a prebuilt kernel for this machine, was closed on
  2026-11-19.
- **Two services failed on every boot** on 2026-11-18 and the system
  reported itself as degraded. See the table below; it was not re-checked on
  2026-11-19.
- **Wi-Fi associates on a WPA2 network** (2026-11-19) after its driver is
  loaded by hand.
  **Bluetooth does not work**: the driver finds no calibration blob. See
  [Booting on the 27.2 stub](booting-27-2-stub.md#known-problems).
- **Bug A below is not seen on 27.2.** The ESP mounts from the identifier in
  `fstab`, and the `apple_wdt` step in the initrd succeeds.
- **The desktop is software-rendered** and very slow; there is no GPU
  driver for the 27.2 firmware.
- **Suspend is untested** on 27.2. The advice under "Suspend is disabled"
  still applies.

| Failing service | Effect | Cause |
|---|---|---|
| Vendor firmware service | On 2026-11-18 the Wi-Fi and Bluetooth drivers stayed held back. Whether this is why the Wi-Fi driver still has to be loaded by hand is not established | The script loads a driver for the always-on processor, which this kernel does not have and which the loader disables on this board, and it stops there. A change to the script passes its unit test but has not been booted |
| Firewall | No packet filter is active | The test kernel lacks a netfilter match module the NixOS firewall always uses. This is a kernel configuration fault, not an image fault |

## How it booted on 26.6.2

The proven route keeps the run-59 Stage 2 live, uses a GRUB one-shot to select
the NixOS BLS entry, and boots the NixOS system/root image with the run-59
kernel. It does not trial a candidate Stage 2. Source:
`private evidence receipt`,
`private evidence receipt`, and `private evidence receipt`.

The last recorded NixOS source pin is
`techmore/nixos-aurora:j773g-nixos-debug-20261112@6a6f297281a4436e409b92705b6ad9dbb8d703bf`
with sanitized head `e2f3a3a04e0b`. Its exact current qualification is
**unverified**; `private evidence receipt` says the kernel owner is qualifying a newer
candidate.

## Bug A: image closure and ESP identity

The first image had an empty/mismatched modules tree while requiring a fatal
`apple_wdt` modprobe, and its filesystem configuration named an ESP UUID that
did not exist on the installed target. The installed ESP identifier did not match the identifier baked into the image. Prefer a stable label or PARTUUID and make an
optional ESP mount non-fatal. Source:
`private evidence receipt` and
`private evidence receipt` “Bug A.”

## Bug B: loader state, not NixOS

A failed Aura trial left a `spent` marker and `.next` payload; a later plain
path then stopped before GRUB. Clearing both restored the known-good chain.
This is a loader recovery-state bug, not evidence against the NixOS root.
Source: `private evidence receipt` and
`private evidence receipt` “Bug B.” Independent causality of either leftover is
**unverified**.

## Suspend is disabled

Until J773g suspend/resume is qualified, the image must set logind idle and
suspend-key actions to `ignore`, configure the desktop never to sleep, and
disable systemd sleep targets. The observed disappearance after idle is only
**probably** suspend; its journal cause is unverified. Source:
`private evidence receipt`, “NixOS LKG left running went unreachable.”

## Image recipe checklist

1. Build the exact pinned kernel with `CONFIG_ARM64_CTXR_EXECMEM=y` and a
   complete matching `/lib/modules/<release>` closure.
2. Create system/root and boot artifacts, verify their hashes, and identify
   filesystems by the actual target label/PARTUUID (record the ESP UUID only
   as evidence, not a portable recipe constant).
3. Install a BLS entry without replacing the saved run-59 default.
4. Disable idle suspend/hibernate as above.
5. Boot through the live Stage 2 plus a GRUB one-shot under `trial-gate.py`;
   verify KDE, network, storage, kernel release and modules, then postflight
   back to run 59.

Sources: `private evidence receipt`,
`install-from-arm-build-host.sh`, `prepare-one-shot-on-run59.sh`, and `RESULT.md` in
that run directory. These scripts may evolve; the receipt remains the
authority for the proven image.
