# Booting on the 27.2 stub

> **Status on 2026-11-19: open pull requests, nothing merged upstream.**
> Everything on this page was run on one M4 Mac mini (J773g) with locally
> built loader and kernel images, on 2026-11-18 and 2026-11-19. The changes
> are now open pull requests in the Aurora project, except the firmware-27
> display commits, which are not yet in any pull request. None of it is
> merged, and none of it is upstream support for this machine. Treat it as a
> lab record. The pull requests are listed in
> [Upstream status](upstream-status.md).

## Summary

With the macOS 27.2 (build 26B123) boot stub, the J773g now boots Linux
unattended through the normal chain (iBoot, Aura Stage 1, Stage 2 on the ESP,
U-Boot, GRUB, NixOS) to SSH and to a KDE desktop on HDMI.

Working, with the open pull requests applied: 10 CPUs, NVMe, 2.5 Gb/s
Ethernet, a USB keyboard and mouse on the front ports at USB 2, and a rear
port as a USB device at USB 2. HDMI video works at 3840x2160, 60 Hz on one
monitor and at 1080p on another, and needs additional firmware-27 display
commits that are not yet in any pull request. Wi-Fi associates on a WPA2
network after its driver is loaded by hand.

Not working or not done: GPU acceleration (the desktop is drawn in software
and is very slow), Bluetooth, HDMI hot-plug after boot, HDMI audio,
Thunderbolt, SuperSpeed on the front ports, and suspend. Thermal management
is partial. The SSD controller sometimes fails to start on a reboot; see
[Known problems](#known-problems).

Until 2026-11-15 this machine ran on a 26.6.2 (25G83) stub, which is what the
other pages of this reference describe. After the stub was reinstalled on
27.2, no kernel we had stayed up and there was no display output at any stage.

## What a "stub" is, and why its version matters

Linux on Apple Silicon is started from a small macOS installation (the stub).
The stub fixes which Apple boot firmware and coprocessor firmware the machine
loads before Linux starts. The SSD controller, the display coprocessor and
the GPU each run Apple firmware of that version, and Linux has to speak the
protocol of that exact version. Moving the stub from 26.6.2 to 27.2 changed
all of them at once.

## Three causes

### 1. The kernel's firmware check accepted only 26.6.2

**Symptom.** Every kernel reset the machine 2 to 4 seconds after it started.
This included two older kernels that were known to boot on 26.6.2, so it was
not a new kernel change.

**Cause.** The M4 platform code in our kernel stack compares the firmware
version the loader reports (`/chosen/asahi,os-fw-version`) against a fixed
value, and for this board it accepted exactly `26.6.2`. When the check fails,
the kernel skips the step that records which power domains belong to firmware
(the PMGR ownership policy), and then manages power domains it must leave
alone.

**Confirmation.** A copy of the same kernel image with the version string
patched from `26.6.2` to `27.2` stayed up. A kernel then built from source
with the check widened for this board on 27.2 behaved the same way, and
booted the installed system:

```text
smp: Brought up 1 node, 10 CPUs
t8132-platform-handoff: supplied 0 platform nodes, 137 PMGR policy properties
```

**Now.** The change is open as [linux #2317](https://github.com/aurora-silicon/linux/pull/2317). It does not add 27.2 to the
check: it removes the version test, so the PMGR policy is applied on any
firmware. The pull request records two boots on this machine on 2026-11-19,
about five minutes each, with a plain kernel command line (none of the
bring-up parameters). They ran on an earlier commit of the branch than its
current head.

**Limits.** The panic message was never captured, because memory is cleared
across the reset, so which access was fatal is unknown. The changed kernel has
not been booted on 26.6.2, and no other M4 board was tested.

### 2. The display firmware protocol changed on 27.2

**Symptom.** No HDMI picture from iBoot, from the loader or from Linux.

The J773g's HDMI port is driven by the display coprocessor (DCP), which runs
Apple firmware. The 26.6.2 path described in [HDMI/display](hdmi-display.md)
did not run on 27.2. Three layers needed changes:

| Layer | What it did on 27.2 before | What changed in the 2026-11-18 local build |
|---|---|---|
| Loader (Stage 2) | Refused the M4 display handoff for this board on anything but 26.6.2 and kept only the boot framebuffer | Admits the J773g on 27.2 and publishes the display coprocessor's memory regions (its log region, the frontend segment and the PIODMA segment) in the device tree, leaving the IOMMU tables untouched |
| Kernel IOMMU driver (DART) | Left the display IOMMUs alone on any firmware but 26.6.2 | Accepts the 27.2 handoff for the Mac mini's main DCP and preserves the firmware's existing mappings without writing to them |
| Kernel display driver | Accepted only the 26.6.2 handoff and stopped otherwise | Uses the firmware 27 path with the 27.2 call table. It also needs two small 27.2 callbacks written by other contributors, which are part of an open upstream pull request, and an existing upstream commit that opens the display-port controller and ports |

What moved, as found on the hardware: the firmware's display-port service is
not where the driver first looked. The second attempt attached to the main
DCP and started the framebuffer service on the 27.2 call table, but no port
service was announced on endpoint `0x2c`. The port service is on
endpoint `0x2a` and its controller on `0x24`. With those, the output came up:

```text
apple-dcp 30ae00000.dcp: firmware-compat 27.2.0: firmware 27 path (t8132_fw27=1)
apple-dcp 30ae00000.dcp: AFK[ep:24]: new service AppleDCPDPTXController on channel 9
apple-dcp 30ae00000.dcp: AFK[ep:2a]: new service AppleDCPDPTXRemotePort on channel 1
apple-dcp 30ae00000.dcp: dcp_hotplug() connected:1 valid_mode:0 nr_modes:41
```

The connector reports as connected with 41 modes and the compositor scans out
at 3840x2160, 60 Hz. The loader half and the kernel half only work together:
either one alone gives no picture. On 2026-11-19 a second monitor ran at
1080p.

**Now.** The loader half was reworked and is open as [aura #199](https://github.com/aurora-silicon/aura/pull/199), which hands
off the HDMI DCP and its display frontend through the shared M4 DCP path;
[aura #258](https://github.com/aurora-silicon/aura/pull/258) was folded into it and closed. It no longer admits a board or an
OS version by name: it selects by what the Apple device tree describes. The
kernel display work it pairs with is [linux #1192](https://github.com/aurora-silicon/linux/pull/1192) (open). The firmware-27
kernel commits described in the table are not yet in any pull request; the
plan is to add them to #1192.

**Limits.** A monitor attached after boot is not picked up (no hot-plug).
Mode changes, display power management and suspend are untested. The
connector exposes no EDID; the mode list comes from the
firmware. HDMI audio does not work. The 26.6.2 path has not been re-tested
with these changes present. There is still no picture before Linux starts
(nothing from iBoot or the loader on 27.2).

### 3. The installed image carried a kernel without those changes

A locally patched test build of the installer wrote a valid 27.2 stub, but
the NixOS image it installed contains a prebuilt kernel made for the 26.6.2
stub. That kernel cannot stay up on 27.2 (cause 1) and has no 27.2 display
code (cause 2). The installed system reached the desktop only after the
kernel image on the boot partition was replaced by hand with one that carries
the changes above. The Stage 2 on the ESP was also replaced, during the SSD
controller investigation described next; whether that was needed is not
established.

So a fresh install on 27.2 does not boot by itself today. The image has to be
rebuilt around a kernel that includes both changes; that rebuild has not been
done. More on the image in [NixOS](nixos.md).

### A suspected fourth cause that was withdrawn

On the first boots after the install, the loader could not start the SSD
controller, so those boots never reached the kernel. That looked like a 27.2
incompatibility, and a set of loader changes appeared to fix it. A later
isolation test showed the unchanged loader starting the controller 5 times
out of 5, so those changes are not credited with anything. The intermittent failure that remains is listed under
[Known problems](#known-problems).

## How it was debugged

At the start there was no display, no network and no log: the machine reset
within seconds or stopped silently. Four things made it tractable.

1. **Reading the loader's log over USB-C.** When Stage 2 cannot continue it
   stays in a debug mode and appears as a USB serial device on a second
   computer connected to one of the Mini's USB-C ports. Its console log can be
   read from there. This was missed at first: an early 60-second watch for
   USB devices saw nothing, and the conclusion that no loader log was
   available was wrong.

2. **Booting test kernels from the loader.** Over the same link, the loader
   can be handed a kernel, device tree and command line directly, with no
   disk or boot menu involved. Each experiment took about a minute. With no
   log available, the measurement was the time until the loader reappeared on
   USB (the machine had reset). Changing `panic=5` to `panic=30` moved the
   reset from 18 to 46 seconds, which showed a real kernel panic rather than
   a watchdog. Disabling individual drivers one at a time did not move it.

3. **A tiny log pump as PID 1.** Memory is cleared across a reset on this
   machine, so the usual crash-log mechanisms recover nothing. A small static
   program, run as the first process from an initramfs, copies the kernel log
   (`/dev/kmsg`) to numbered files on the boot (EFI) partition, and the loader
   can read those files back over USB. It could not capture the original
   panic, because the disk is not up that early. Once the kernel stayed up it
   returned complete kernel logs from boots that had no display and no login,
   which is what the display work ran on.

4. **One-shot boots with automatic fallback.** Later tests ran without anyone
   at the machine. The loader has a one-shot marker: a trial Stage 2 is used
   for a single boot under a watchdog, and the next boot returns to the
   installed one (see [Boot chain and recovery](boot-chain.md)). GRUB's
   `next_entry` does the same for a kernel. Each test kernel also ran with a
   short panic timeout and a small unit that restarts the machine after 150
   seconds unless it is cancelled over SSH. A failed test therefore returned
   to the known-good system by itself, and the default boot path was never
   replaced for a test.

A useful shortcut: before changing any source, the firmware check was tested
by patching the version string in a copy of the kernel image. That confirmed
the cause in minutes; the source change followed.

## Support matrix on the 27.2 stub

State on 2026-11-19, one machine, local builds of the open pull requests
plus the firmware-27 display commits. The boot, Ethernet, HDMI, GPU, Wi-Fi,
Bluetooth, USB and Thunderbolt rows were updated on 2026-11-19. The other
rows are from 2026-11-18 on the earlier local build and were not re-checked.

| Function | Status | Notes |
|---|---|---|
| Unattended boot through the normal chain | Works | Reaches SSH. See the SSD controller entry under Known problems |
| CPUs | Works | 10 online; frequency scaling and idle states active |
| NVMe | Works | |
| Ethernet | Works | 2.5 Gb/s. [linux #1898](https://github.com/aurora-silicon/linux/pull/1898) (open), tested on two units (the second by DjDeveloperr, on 26.6.2) |
| HDMI video | Works | 3840x2160 at 60 Hz on one monitor, 1080p on another. Needs firmware-27 display commits that are not yet in any pull request |
| HDMI EDID | Not working | The connector reports no EDID |
| HDMI hot-plug | Not working | A monitor attached after boot is not picked up |
| HDMI mode change, display power management | Untested | |
| HDMI audio | Not working | No sound card |
| GPU acceleration | Not working | The loader disables the GPU on 27.2; the desktop is software-rendered and very slow |
| Wi-Fi | Partly | Associates on a WPA2 network after the driver is loaded by hand. It does not load by itself; the reason is not known |
| Bluetooth | Not working | The Apple device tree carries no calibration blob for the driver. DjDeveloperr's [linux #2235](https://github.com/aurora-silicon/linux/pull/2235) (open) addresses this; it has not been tested on this machine |
| Rear USB-C ports, USB 2 host | Worked on two ports on 2026-11-18 | Keyboard and mouse enumerated on the earlier local build. The third port held the debug cable. A confirming test on the current build is still to do |
| Rear USB-C port as a USB device | Works at USB 2 speed | With [linux #1702](https://github.com/aurora-silicon/linux/pull/1702) (open) |
| Front USB-C ports (internal two-port hub) | Works at USB 2 speed | USB keyboard and mouse |
| USB 3 speeds | Not working or untested | Front: SuperSpeed does not work. Rear: nothing above USB 2 is claimed |
| Thunderbolt / USB4 | Not working | [linux #1703](https://github.com/aurora-silicon/linux/pull/1703) is a draft |
| Headphone jack, speaker | Not working | No sound card |
| Sensors and fan speed reading | Works | |
| Fan control, thermal zones | Not present | Thermal management is partial: the PMP firmware starts, its temperature sensors return errors |
| Suspend | Untested | |
| Clock (RTC), watchdog | Works | |
| Power button | Untested | |
| Always-on processor (AOP), camera processor (ISP) | Disabled on 2026-11-18 | The earlier local loader disabled both. [aura #113](https://github.com/aurora-silicon/aura/pull/113) (open) hands the AOP off and applies a rule for an absent ISP, selected by ADT property |

## Known problems

- **The SSD controller sometimes fails to start on a reboot.** This has been
  seen 8 times so far. When it happens the loader finds no disk, stops in its
  debug mode and waits; a reboot recovers it. It needs a fix in Stage 1,
  which is not written. A machine that must come back unattended is not
  reliable yet.
- **The desktop is very slow.** There is no GPU driver for this firmware, so
  the compositor draws on the CPU. One measurement at 4K showed about one
  frame per second, which makes input look dead.
- **Wi-Fi needs its driver loaded by hand.** After that it associates on a
  WPA2 network. Why the driver does not load by itself is not known.
- **No Bluetooth.** The driver needs a calibration blob that the Apple device
  tree on this machine does not carry. DjDeveloperr's [linux #2235](https://github.com/aurora-silicon/linux/pull/2235) (open) loads
  it from a firmware file instead; it has not been tested on this machine.
- **HDMI hot-plug and HDMI audio do not work.** A monitor has to be attached
  before boot.
- **The firewall service fails**, because the test kernel lacks one netfilter
  module the NixOS firewall needs. The test system runs with no packet filter.
- **Bring-up kernel options.** [linux #2317](https://github.com/aurora-silicon/linux/pull/2317)
  booted twice without them (the ones that keep unused clocks and power
  domains on), about five minutes each. A longer run has not been done.
- **Cold starts are thinly covered.** Most test boots were warm restarts.
- **Nothing here was re-tested on a 26.6.2 stub on this machine.**
  DjDeveloperr runs [linux #1192](https://github.com/aurora-silicon/linux/pull/1192) as the default boot on his unit on 26.6.2.

## Where the changes are

The loader and kernel changes are open pull requests in the Aurora project,
listed in [Upstream status](upstream-status.md). The firmware-27 display
commits are the exception: they are not yet in any pull request. Nothing on
this page should be read as merged, accepted or supported upstream.
