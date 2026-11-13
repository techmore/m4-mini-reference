# M4 J773g/T8132 invariants

> Generated from `invariants.yaml`; do not hand-edit.

20 fail-closed requirements shared across the M4 stack.

## Last cross-repository audit

| Invariant | linux | linux-aurora | fedora-aurora | nixos-aurora | alarm-aurora | omarchy-mac-pkgs | aura | u-boot | aurora-installer | mesa |
|---|---|---|---|---|---|---|---|---|---|---|
| `kernel-ctxr-execmem` | FAIL | FAIL | PASS | PASS | PASS | FAIL | N-A | N-A | N-A | N-A |
| `kernel-topic-order` | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A |
| `kernel-pcie-pmgr-seven` | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A |
| `kernel-package-modules` | N-A | N-A | PASS | PASS | PASS | FAIL | N-A | N-A | N-A | N-A |
| `aura-post-m4-ans-takeover` | N-A | N-A | PASS | FAIL | FAIL | FAIL | FAIL | N-A | FAIL | N-A |
| `aura-hdmi-rail-positive-read` | N-A | N-A | N-A | N-A | N-A | N-A | FAIL | N-A | N-A | N-A |
| `aura-hpd-delay-replay` | N-A | N-A | N-A | N-A | N-A | N-A | PASS | N-A | N-A | N-A |
| `aura-bounded-dcp-polling` | N-A | N-A | N-A | N-A | N-A | N-A | FAIL | N-A | N-A | N-A |
| `aura-sid23-locked-dart-dva` | FAIL | FAIL | N-A | N-A | N-A | N-A | PASS | N-A | N-A | N-A |
| `aura-ans-takeover-scope` | N-A | N-A | N-A | N-A | N-A | N-A | PASS | N-A | N-A | N-A |
| `boot-clear-trial-state` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | FAIL | N-A | PASS | N-A |
| `boot-aura-try-two-line` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | FAIL | N-A | FAIL | N-A |
| `boot-grub-visible` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | N-A | N-A | FAIL | N-A |
| `boot-promote-passing-fix` | N-A | N-A | N-A | N-A | N-A | N-A | FAIL | N-A | PASS | N-A |
| `uboot-comparison-unverified` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | N-A | FAIL | FAIL | N-A |
| `nixos-esp-stable-nofail` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | N-A | N-A | FAIL | N-A |
| `nixos-no-idle-suspend` | N-A | N-A | PASS | FAIL | FAIL | FAIL | N-A | N-A | FAIL | N-A |
| `nixos-apple-wdt-nonfatal` | N-A | N-A | FAIL | FAIL | FAIL | FAIL | N-A | N-A | FAIL | N-A |
| `display-dcp-handoff-topic` | FAIL | FAIL | FAIL | FAIL | FAIL | FAIL | N-A | N-A | N-A | N-A |
| `ops-preserve-toolchains-inputs` | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A | N-A |

## Registry

### kernel-ctxr-execmem

- **Symptom:** Only one CPU reaches EL1; expected 10 CPUs.
- **Cause:** Kernel executable memory is outside the iBoot CTXR window.
- **Required state:** CONFIG_ARM64_CTXR_EXECMEM=y in every J773g distro and Omarchy kernel config.
- **Applies to:** linux, linux-aurora, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs
- **Where:** linux #985 / every distro kernel config
- **Upstream:** merged; carried by distro recipes
- **Evidence:** 2026-11-07 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `kernel_config`
- **Fix:** Enable CONFIG_ARM64_CTXR_EXECMEM and rebuild the binary kernel.

### kernel-topic-order

- **Symptom:** PMP declarations conflict or DCP reservations disappear after stack replay.
- **Cause:** g16-pmp was applied before dcp-handoff/platform-io.
- **Required state:** dcp-handoff precedes platform-io and g16-pmp.
- **Applies to:** linux, linux-aurora
- **Where:** aurora/integration/TOPICS.tsv
- **Upstream:** carried
- **Evidence:** 2026-11-10 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `kernel_order`
- **Fix:** Replay topics in the documented linear order.

### kernel-pcie-pmgr-seven

- **Symptom:** PCIe/NIC disappears on the newer-base DTB.
- **Cause:** Seven PMGR provider domains remain disabled.
- **Required state:** All seven reviewed PMGR domains have status=okay.
- **Applies to:** linux, linux-aurora, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs
- **Where:** aurora/overlays/pmgr-status-okay
- **Upstream:** carried
- **Evidence:** 2026-11-10 first seen; 2026-11-12 last verified; `private evidence receipt`
- **Machine check:** `dtb_property`
- **Fix:** Apply the pmgr-status-okay overlay to the final DTB.

### kernel-package-modules

- **Symptom:** FATAL: Module apple_wdt not found; sysroot.mount fails.
- **Cause:** Binary kernel package has an empty modules tree.
- **Required state:** Package lib/modules with matching metadata.
- **Applies to:** fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs
- **Where:** binary kernel package recipe
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `modules_tree`
- **Fix:** Install and package the complete modules tree (Bug A).

### aura-post-m4-ans-takeover

- **Symptom:** nvme@40000 err=-19; U-Boot sees no NVMe.
- **Cause:** Live post-M4 ANS was not taken over before a non-Linux payload.
- **Required state:** Aura includes f2a8b52b post-M4 ANS takeover.
- **Applies to:** aura, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs, aurora-installer
- **Where:** techmore/aura f2a8b52b
- **Upstream:** carried
- **Evidence:** 2026-11-13 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `aura_commit`
- **Fix:** Rebase/cherry-pick the ANS takeover before composing Stage 2.

### aura-hdmi-rail-positive-read

- **Symptom:** gP0f/gP1a rail votes are skipped despite a successful ADT read.
- **Cause:** Positive byte count was treated as failure.
- **Required state:** Positive ADT read length is success for J773g HDMI rail keys.
- **Applies to:** aura
- **Where:** aura J773g HDMI rail code
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Accept the positive byte count and issue both rail votes.

### aura-hpd-delay-replay

- **Symptom:** HDMI remains dark after handoff.
- **Cause:** HPD is not asserted and replayed after the converter settles.
- **Required state:** J773g HPD assert plus replay after 20-25 ms.
- **Applies to:** aura
- **Where:** aura J773g DCP HDMI path
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Restore the bounded settle delay and HPD replay.

### aura-bounded-dcp-polling

- **Symptom:** Boot hangs forever in DCP polling.
- **Cause:** A firmware poll has no deadline.
- **Required state:** Every DCP poll is bounded and fails closed.
- **Applies to:** aura
- **Where:** aura DCP/RTKit path
- **Upstream:** carried
- **Evidence:** 2026-11-05 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Add a measured deadline and propagate timeout failure.

### aura-sid23-locked-dart-dva

- **Symptom:** inherited SID23 domain not qualified; v5 also broke USB and GPU.
- **Cause:** Locked-DART inherited DVAs/reservations were rewritten or omitted.
- **Required state:** Preserve and qualify SID23 DVA mappings for locked DARTs.
- **Applies to:** aura, linux, linux-aurora
- **Where:** aura dcp-handoff/DART code
- **Upstream:** carried
- **Evidence:** 2026-11-11 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Restore the v4 inherited locked-DART DVA handling.

### aura-ans-takeover-scope

- **Symptom:** USB/GPU regress while fixing NVMe.
- **Cause:** ANS takeover is applied outside the post-M4 non-Linux payload scope.
- **Required state:** Take over only live post-M4 ANS before non-Linux payloads.
- **Applies to:** aura
- **Where:** aura payload dispatch
- **Upstream:** carried
- **Evidence:** 2026-11-13 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Narrow the takeover predicate; do not broaden coprocessor ownership.

### boot-clear-trial-state

- **Symptom:** Plain boot hangs before GRUB with spent + .next present.
- **Cause:** Stale Aura trial state survived a trial.
- **Required state:** Clear spent, .next and metadata after every trial.
- **Applies to:** aura, aurora-installer, all-j773g-dist-images
- **Where:** trial gate and installer recovery
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Run postflight cleanup and read back absence before another boot.

### boot-aura-try-two-line

- **Symptom:** Candidate is ignored or wrong payload is selected.
- **Cause:** aura-try marker is not exact two-line format.
- **Required state:** armed newline relative-stage2-path newline.
- **Applies to:** aura, aurora-installer, all-j773g-dist-images
- **Where:** trial gate/installer
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Write exactly the documented two newline-terminated records.

### boot-grub-visible

- **Symptom:** No recoverable boot menu is shown.
- **Cause:** GRUB auto-hide or zero timeout is enabled.
- **Required state:** menu_auto_hide unset; GRUB_TIMEOUT_STYLE=menu; timeout at least 5 seconds.
- **Applies to:** aurora-installer, all-j773g-dist-images
- **Where:** every J773g distro image/installer
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `grub_visible`
- **Fix:** Disable auto-hide and regenerate grub.cfg with a five-second menu.

### boot-promote-passing-fix

- **Symptom:** Next boot silently returns to a broken live Stage 2.
- **Cause:** A proven trial was not promoted.
- **Required state:** Promote an exact passing fix to live immediately.
- **Applies to:** aura, aurora-installer
- **Where:** trial-gate postflight
- **Upstream:** carried
- **Evidence:** 2026-11-13 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Atomically install the passing artifact and retain a rollback copy.

### uboot-comparison-unverified

- **Symptom:** Test A returned to macOS before GRUB, but changed both Aura and U-Boot.
- **Cause:** Unverified: the failed Stage 2 lost Aura's f2a8b52b ANS takeover; e22da7f0 is a raw SHA-256 of a TEST ONLY debug build, not a U-Boot commit.
- **Required state:** Run two boots with the identical Aura, DTB, kernel and Stage 2 layout, changing only U-Boot aurora-wip 25cbb0f0 versus u-boot #48 cfebc31d.
- **Applies to:** u-boot, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs, aurora-installer
- **Where:** aurora-silicon/u-boot aurora-wip 25cbb0f0 and PR #48 cfebc31d
- **Upstream:** unverified; #48 open
- **Evidence:** 2026-11-13 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `uboot_two_boot_comparison`
- **Fix:** Do not pin or blame either build from Test A; record the two single-variable hardware receipts first.

### nixos-esp-stable-nofail

- **Symptom:** sysroot.mount fails; ESP UUID [redacted filesystem ID] does not exist.
- **Cause:** Image hard-codes a per-filesystem UUID and makes it required.
- **Required state:** ESP by label or PARTUUID with nofail.
- **Applies to:** aurora-installer, all-j773g-dist-images
- **Where:** every J773g distro image/installer
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `nixos_content`
- **Fix:** Use a stable label/PARTUUID and make ESP failure non-fatal.

### nixos-no-idle-suspend

- **Symptom:** An idle J773g becomes unreachable.
- **Cause:** Default idle suspend runs on unsupported M4 resume.
- **Required state:** Disable idle suspend/hibernate.
- **Applies to:** all-j773g-dist-images
- **Where:** every J773g distro image
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `nixos_content`
- **Fix:** Disable logind, desktop, and systemd sleep paths.

### nixos-apple-wdt-nonfatal

- **Symptom:** Boot aborts when modprobe apple_wdt returns nonzero.
- **Cause:** Watchdog modprobe is a fatal precondition.
- **Required state:** apple_wdt modprobe is optional/non-fatal.
- **Applies to:** all-j773g-dist-images
- **Where:** every J773g distro image
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `nixos_content`
- **Fix:** Prefix the unit command with '-' or otherwise ignore absence.

### display-dcp-handoff-topic

- **Symptom:** Display stays on simpledrm; Apple DCP rejects SID23.
- **Cause:** Aura framebuffer ownership lacks kernel dcp-handoff reservations.
- **Required state:** Kernel includes dcp-handoff when Aura owns the framebuffer.
- **Applies to:** linux, linux-aurora, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs
- **Where:** linux topic stack
- **Upstream:** carried
- **Evidence:** 2026-11-11 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `content`
- **Fix:** Carry the dcp-handoff topic with the Aura Stage 2.

### ops-preserve-toolchains-inputs

- **Symptom:** Build cannot start after cleanup.
- **Cause:** Cleanup deleted a compiler closure or gate input.
- **Required state:** Cleanup denylist protects toolchains and gate inputs.
- **Applies to:** linux, linux-aurora, fedora-aurora, nixos-aurora, alarm-aurora, omarchy-mac-pkgs, aura, u-boot, aurora-installer, mesa
- **Where:** workspace cleanup scripts
- **Upstream:** carried
- **Evidence:** 2026-11-12 first seen; 2026-11-13 last verified; `private evidence receipt`
- **Machine check:** `cleanup_guard`
- **Fix:** Protect registered roots; refuse cleanup when inputs are unresolved.
