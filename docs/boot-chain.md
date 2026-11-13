# Boot chain and recovery

## The chain

```text
iBoot -> Aura Stage 1 -> ESP Stage 2 (Aura + J773g DTB + gzip U-Boot)
      -> U-Boot EFI -> GRUB -> BLS entry -> Linux kernel
```

Stage 1 is enrolled and normally loads the configured Stage 2 from the ESP.
Stage 2 parses its embedded DTB, hands hardware state to U-Boot, and U-Boot
starts GRUB. GRUB's one-shot `next_entry` selects a BLS entry without changing
the saved run-59 default. Sources: `private evidence receipt` (Stage-1/Stage-2
contracts), `private evidence receipt`, and
`private evidence receipt` on
notes branch `j773g/boot-hdmi-20261112@bec15a4d6106`.

## J773g partition map

| GPT entry | Role | Identity | State |
|---|---|---|---|
| p1–p3 | Apple iBoot, macOS, Aura stub APFS | fixed GPT GUIDs in the private receipt | unchanged |
| p4 | Aura ESP | PARTUUID `d963b95f-801f-42d2-afa0-b4ddbc427611` | mounted `/boot/efi` |
| p5 | Fedora `/boot` | UUID `fc8e1aa7-1a0d-471e-b210-3d2bde0d77fd` | unchanged |
| p6 | Fedora root | UUID `fd7df531-76ae-44e5-a8be-ac6ed5f28ffa`, subvolume `root` | run 59 |
| p7 | retired Omarchy APFS stub | PARTUUID `c25f590a-9fd6-4e13-a8e9-6a4b762edb5a` | unchanged |
| p8 | retired Omarchy ESP | former PARTUUID `bba7efa2-57b4-4b71-a1a2-609b9ca2592f` | **deleted 2026-11-13; 500 MiB free** |
| p9 | NixOS `/boot` | UUID `7179b1dc-ff4c-44e9-8c77-114d3135455d` | unchanged |
| p10 | NixOS root | UUID `04aa1ded-e761-4627-b95c-5d9d937f7016`, subvolume `root` | unchanged |
| p11 | Apple RecoveryOSContainer | PARTUUID `0954d689-6d86-4c66-87fe-cd6d5852465d` | unchanged |

The p8 deletion followed a full content inventory, GPT backup, verified compressed
partition image, and exact before/after comparison of every other partition's
start, end, type, and unique GUID. No reboot was required.

## Known-good and current results

| Tuple | Result | Source |
|---|---|---|
| Stage 2 SHA-256 `1838b1f3…`; kernel Image SHA-256 `13d3d966…` | run 59; Fedora/HDMI known-good | `private evidence receipt`; run-59 receipt cited there |
| Stage 2 `ce8675f9…` (Aura `f2a8b52b` plus old run-59 U-Boot) | GRUB visible; Fedora run 59 and NixOS boot | `private evidence receipt`; exact receipt **unverified on origin/main** |
| Aura `f2a8b52b` plus new U-Boot `0ac2fb76` lineage | stops before GRUB | `private evidence receipt` |
| boot-HDMI Aura `7c5ab9a3` plus run-59 DTB/U-Boot | hangs | boot-HDMI run receipt **unverified on origin/main**; composition analysis at the path above |

Hashes shortened with an ellipsis are identifiers, not sufficient artifact
verification. Use the full SHA-256 in the cited receipt before staging.

## Trial gate

1. Keep run 59 as the saved/default BLS entry and the live Stage 2 immutable.
2. Run `private evidence receipt` preflight. It verifies lease,
   clean baseline, recovery, selectors, artifact hashes, and absent stale trial
   state.
3. Arm exactly one selector: an Aura `aura-try` Stage-2 trial **or** GRUB
   `next_entry`, never both. For current NixOS/kernel trials use live Stage 2
   plus the GRUB one-shot.
4. Observe a real down/up transition and identify the resulting OS before
   judging the run.
5. Run postflight; return to two clean run-59 baselines before another arm.

Sources: `private evidence receipt`, `private evidence receipt`, and
`private evidence receipt` entries “Bug B” and “Candidate Stage 2 failed before
GRUB.”

## Recovery and selectors

- `aura-proxy` is an ESP-root selector that opens Aura Stage 1's short proxy
  window. It is not a boot argument on this enrolled J773g path. Remove it
  after the proxy run. Source: `private evidence receipt` F8 and selector table.
- `aura-try` contains these newline-separated records:

  ```text
  armed
  <relative-stage2-path>
  ```
 Aura rewrites it to
  `spent` before reading the candidate, so the next boot returns to the normal
  path. Source: `private evidence receipt`, citing Aura
  `src/chainload.c:334-339` and `rust/src/trial.rs:4-7`.
- Recovery uses the documented proxy/macOS read-only path and the immutable
  known-good Stage 2. Do not warm-retry a DCP/PMP/RTKit fault. Source:
  `private evidence receipt` and `private evidence receipt`.

### The stale `spent`/`.next` trap

After a failed Aura trial, both the `spent` marker and its
`m1n1/boot-ctxr.bin.next` can remain. On J773g that pair made a later plain
known-good path hang before GRUB. Preflight and postflight must require absent
trial state and an empty GRUB `next_entry`; recovery requires two clean run-59
baselines before re-arming. Source:
`private evidence receipt` and
`private evidence receipt` “Bug B.” Which member of the pair is independently
causal is **unverified**.
