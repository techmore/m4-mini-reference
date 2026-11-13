# NixOS on J773g

## How it boots today

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
