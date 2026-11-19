# M4 Mac mini (J773g / T8132) reference


This is the single maintained reference for the M4 Mac mini bring-up while
the work is reconciled and prepared for upstream. Facts cite repository
paths, public fork/upstream commits, or run receipts. Anything without such
evidence is marked **unverified**.

## Current status

**Tested locally, not yet upstream.** On the macOS 27.2 (26B123) boot stub the
J773g boots Linux unattended to a 4K HDMI desktop, with 10 CPUs, NVMe, 2.5 Gb
Ethernet and USB 2. GPU acceleration, Wi-Fi, Bluetooth, audio and
Thunderbolt/USB4 do not work, and the SSD controller sometimes fails to start.
This was measured on one machine with local builds; none of it is merged in
the upstream projects. Details, the support matrix and known problems are in
[Booting on the 27.2 stub](booting-27-2-stub.md).

The other pages were written for the 26.6.2 (25G83) stub this machine used
until 2026-11-15, and say so where 27.2 differs.

## Topics

- [Booting on the 27.2 stub](booting-27-2-stub.md) — what broke on the 27.2
  firmware, the three causes, how it was debugged, the support matrix, and
  known problems.
- [M4 invariants](invariants.md) — cross-repository must-haves and the latest
  machine-check matrix.
- [Boot chain and recovery](boot-chain.md) — stages, tuples, trial gate,
  proxy recovery, and the stale trial-state trap.
- [HDMI/display](hdmi-display.md) — concise Linux bring-up order and exact
  code pointers.
- [Kernel configuration](kernel-config.md) — SMP config, topic order, and the
  PMGR overlay root cause.
- [NixOS](nixos.md) — current boot path, Bugs A/B, ESP identity, suspend, and
  image recipe.
- [Benchmarks](benchmarks.md) — public cross-OS baseline and qualification status.
- [Upstream status](upstream-status.md) — public carried branches, topic dependencies, and drift.

This public edition contains only scrubbed, non-sensitive material. The evidence ledger and maintenance tooling remain private.
