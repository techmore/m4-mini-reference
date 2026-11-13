# M4 Mac mini cross-OS benchmarks

## Qualification status

The benchmark harness uses the same portable workloads on macOS and Aurora,
three independent timed runs, and reports the median with the minimum and
maximum. Tool and model versions are pinned, inputs are hashed, and each run
records OS, kernel, compiler, CPU topology, thermal state, and available power
telemetry.

The current qualified public comparison is intentionally incomplete. A stock
macOS baseline and an older Aurora-kernel STREAM run are available, but a
complete NixOS Aurora pass has not yet been admitted through the recovery-safe
boot gate. Missing cells are not estimates.

The J773g under test is a Mac16,10 with an Apple M4 (10 CPU cores: 4
performance and 6 efficiency; 10 GPU cores), 16 GB unified memory, and a 256
GB internal Apple SSD. A planned simultaneous A/B with a second macOS Mini is
same-SKU evidence only after those CPU, memory, and storage characteristics
are confirmed to match.

| Workload | macOS | Aurora Linux evidence | Ratio (Aurora/macOS) |
|---|---:|---:|---:|
| STREAM Triad, 1.5 GiB | 93.97 [93.62, 94.35] GB/s | 95.35 [95.05, 95.68] GB/s | 1.015x |
| 7-Zip, one thread | 11,606 [11,406, 11,725] MIPS | not measured | — |
| 7-Zip, ten threads | 80,924 [79,644, 81,548] MIPS | not measured | — |
| Sequential 1 MiB read, QD1 | 2,610.92 [2,608.54, 2,612.81] MB/s | not measured | — |
| Random 4 KiB read, QD32 | 163,132 [161,919, 163,750] IOPS | not measured | — |

The STREAM result shows no regression in the older Aurora-kernel run: its
median is 1.5% above macOS. It does not qualify NixOS Aurora as a whole. CPU,
compile, storage, GPU, boot-time, and power comparisons remain open until the
same suite completes on the current NixOS image.

## Candidate performance investigations

These are hypotheses to test, not measured NixOS regressions:

1. CPU single-thread performance: verify CLPC/cpufreq residency and sustained
   performance-core frequency under 7-Zip and sysbench.
2. CPU multi-thread performance: compare scheduler placement, capacity data,
   and thermal throttling across the ten cores.
3. GPU and LLM throughput: compare the same llama.cpp revision, model,
   quantization, prompt, and token counts using Metal and Vulkan; glmark2 and a
   macOS-native graphics test are not treated as directly comparable.
4. NVMe: compare scheduler, queue depth, direct-I/O support, and filesystem
   mount options with identical fio job files and scratch-file sizes.
5. Boot time and idle power: separate firmware/loader time from userspace and
   capture Linux hwmon/SMC telemetry only when a supported sensor path exists.

Any verified gap greater than 20% becomes a candidate invariant or tracked
performance ticket with the raw three-run evidence attached.

## Reproduction contract

Use the repository's portable runner and exact pinned inputs. Run macOS first,
then select NixOS only through the one-shot trial gate, inhibit idle sleep for
the session, and return to the saved Fedora baseline with an empty one-shot
selector and no trial files. Never compare best-of-run values: report the
median, range, and relative spread from all three runs.
