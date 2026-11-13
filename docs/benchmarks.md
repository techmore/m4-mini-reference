# M4 Mac mini cross-OS benchmarks

## Qualification status

The benchmark harness uses the same portable workloads on macOS and Aurora,
three independent timed runs, and reports the median with the minimum and
maximum. Tool and model versions are pinned, inputs are hashed, and each run
records OS, kernel, compiler, CPU topology, thermal state, and available power
telemetry.

The current comparison combines the committed 2026-11-07 macOS baseline with
a 2026-11-13 Fedora run-59 pass. NixOS remains unmeasured: its one-shot was
cleared after the mandatory gate refused admission. Missing cells are not
estimates.

A same-day follow-up intended to correct the 7-Zip-version and fio-job
mismatches stopped before measurement: Fedora was reachable and presented its
expected host identity, but rejected the provisioned test-runner
authentication even after one bounded recovery reset.  Consequently there are
still no fair 7-Zip, fair fio, NixOS, or small-model ML numbers.  The values
below remain provisional historical results and were not relabeled as fair.

The J773g under test is a Mac16,10 with an Apple M4 (10 CPU cores: 4
performance and 6 efficiency; 10 GPU cores), 16 GB unified memory, and a 256
GB internal Apple SSD. A planned simultaneous A/B with a second macOS Mini is
same-SKU evidence only after those CPU, memory, and storage characteristics
are confirmed to match.

| Workload | macOS 2026-11-07 | Fedora run 59 | Fedora/macOS |
|---|---:|---:|---:|
| STREAM Triad | 93.97 [93.62, 94.35] GB/s | 92.83 [90.51, 95.33] GB/s | 0.988x |
| 7-Zip, one thread | 11,606 [11,406, 11,725] MIPS | 4,317 [4,316, 4,323] MIPS | 0.372x |
| 7-Zip, ten threads | 80,924 [79,644, 81,548] MIPS | 41,681 [41,562, 41,947] MIPS | 0.515x |
| Sequential read QD1 | 2,610.92 [2,608.54, 2,612.81] MB/s | 336.53 [307.53, 1,160.75] MB/s | 0.129x |
| Random 4 KiB read QD32 | 163,132 [161,919, 163,750] IOPS | 8,735 [8,678, 8,948] IOPS | 0.054x |

STREAM is within 1.2% of macOS. The large 7-Zip and storage gaps are candidate
findings: tool versions and fio scratch sizes still need exact matching before
they become qualified invariants. NixOS, GPU, ML, boot-time, and power cells
remain open.

## ML next run

The bounded 16 GB matrix uses Qwen2.5 0.5B/1.5B/3B, Llama 3.2 1B/3B, and a
Phi-mini-class model in Q4_K_M, plus Q8_0 and F16 for the smallest model.
llama.cpp CPU and Vulkan measure pp512 and tg128 separately; Linux MLX repeats
the same models where supported. Every model and prompt is SHA-256 recorded.

Microbenchmarks cover 512/1024/2048/4096 square GEMM in fp32/fp16/bf16 where
supported, batch-1 GEMV, batched small tensors, and a ResNet-like conv2d.
MobileNetV2, ResNet-50, and whisper tiny/base use fixed hashed inputs. The
macOS counterparts are MLX/Metal, Core ML CPU/ANE/all policies, PyTorch MPS,
and tensorflow-metal. omarchy-ANE results are accepted only with proven J773g
device placement.

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
