# Kernel configuration and topic order

## Required configuration

- `CONFIG_ARM64_CTXR_EXECMEM=y` is required for the SMP J773g kernel. The
  NixOS kernel failure and corrected recipe are recorded in
  `private evidence receipt` and the config at
  `private evidence receipt`.
- `CONFIG_NR_CPUS=64` is the carried known-good value; the machine exposes ten
  CPUs. Sources: `private evidence receipt` and the proven configs under
  `private evidence receipt` (for example
  `native-rear-typec-fix9-20261111/config-fix9-103b4f25`). Whether a smaller
  production value should be upstream is **unverified**.
- Display builds require `CONFIG_DRM_APPLE=y`,
  `CONFIG_DRM_APPLE_DCP_T8132=y`, `CONFIG_APPLE_DART=y`,
  `CONFIG_APPLE_DART_J713_DCP=y`, and `CONFIG_PHY_APPLE_DPTX=y`. Source:
  `private evidence receipt`, display config closure rule.

## Topic dependency order

The linear Linux stack is:

```text
dptx-hdmi-dt -> hdmi-runtime -> dcp-handoff -> platform-io -> g16-pmp
```

The important dependency is that `g16-pmp` is replayed only after
`dcp-handoff` (and the intervening platform-I/O topic). Do not merge topic
heads in arbitrary order. Sources: `private evidence receipt` “Linux stack” and
`private evidence receipt`. Exact current heads are generated in
[upstream-status.md](upstream-status.md).

## PCIe PMGR overlay root cause

The pre-#1171 test DTB wired PCIe, ATC2 and DPTX consumers to PMGR domains
whose nodes had no `status = "okay"`. The #1171 handoff policy correctly
disabled those consumers; the resulting missing PCIe/NIC was a DT defect, not
a #1171 regression. The `pmgr-status-okay` overlay explicitly enables the
seven reviewed domains. The old Air-only early-return patch `e7b68b1d` is a
test workaround and must not be upstreamed. Sources:
`private evidence receipt`,
`private evidence receipt`,
`private evidence receipt`, and `private evidence receipt`
row describing the pre-#1171 DTB.
