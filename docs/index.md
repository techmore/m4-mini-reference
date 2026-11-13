# M4 Mac mini (J773g / T8132) reference


This is the single maintained reference for the M4 Mac mini bring-up while
the work is reconciled and prepared for upstream. Facts cite repository
paths, public fork/upstream commits, or run receipts. Anything without such
evidence is marked **unverified**.

## Topics

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
