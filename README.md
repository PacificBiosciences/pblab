# pblab: PacBio HiFi analysis tools

```text
  .-----------------.         _     _       _
  | o o o o o o o o |   _ __ | |__ | | __ _| |__
  | o o o * o o o o |  | '_ \| '_ \| |/ _` | '_ \
  | o o o o o o * o |  | |_) | |_) | | (_| | |_) |
  | o * o o o o o o |  | .__/|_.__/|_|\__,_|_.__/
  '-----------------'  |_|
     |                   Minutes, not hours.
     '--> HiFi reads ===> align ===> call ===> score
```

Fast tools for HiFi data analysis, tuned to work together: pbweave aligns, pbdv calls small variants, the SV caller calls structural variants, pbhappy and pbtruvari score calls against truth sets. All numbers are for one human genome: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

pbdv, pbhappy and pbtruvari reimplement DeepVariant, hap.py (with RTG Tools vcfeval) and truvari. pblab is not affiliated with or endorsed by Google, Illumina, Real Time Genomics, NVIDIA or the authors of these tools; their names describe what is reimplemented or compared.

## Tools

Fastest time per genome: one AMD EPYC 9755 CPU, 128 cores + SMT (256 vCPU); pbdv with one RTX PRO 6000.

|              Tool              |                  Task                  |        Status       |   60 Gb   |   90 Gb   |
| ------------------------------ | -------------------------------------- | ------------------- | --------- | --------- |
| [pbweave](docs/pbweave.md)     | Alignment, unsorted BAM                | preview             | **21 s**  | **31 s**  |
|                                | Alignment, sorted BAM + BAI            |                     | **29 s**  | **47 s**  |
| [pbdv](docs/pbdv.md)           | Small-variant calling (+ RTX PRO 6000) | preview             | **30 s**  | **33 s**  |
| [SV caller](docs/sv-caller.md) | Structural-variant calling             | Research Experiment | **19 s**  | **26 s**  |
| [pbhappy](docs/pbhappy.md)     | Small-variant benchmarking             | preview             | **2.2 s** | **2.2 s** |
| [pbtruvari](docs/pbtruvari.md) | SV benchmarking                        | preview             | **42 s**  | **39 s**  |

## A genome in the cloud

Reads to sorted BAM, small variants and SVs, one step after the other, on AWS HealthOmics instances. Compute only, on-demand prices, us-east-1 (price list 2026-09-11).

**60 Gb**

|         Step        |             Tool            |     Instance     |     Time     |    Cost   |
| ------------------- | --------------------------- | ---------------- | ------------ | --------- |
| Align + sort        | pbweave                     | omics.c.32xlarge | 30 s         | $0.06     |
| Small variants      | pbdv                        | omics.g6.4xlarge | 2:45 min     | $0.08     |
| Structural variants | SV caller                   | omics.c.32xlarge | 23 s         | $0.05     |
| **Total**           |                             |                  | **3:38 min** | **$0.19** |
| Old pipeline        | pbmm2, DeepVariant, sawfish | omics.c.32xlarge | 44:17 min    | $5.42     |

**90 Gb**

|         Step        |             Tool            |     Instance     |     Time     |    Cost   |
| ------------------- | --------------------------- | ---------------- | ------------ | --------- |
| Align + sort        | pbweave                     | omics.c.32xlarge | 47 s         | $0.10     |
| Small variants      | pbdv                        | omics.g6.4xlarge | 3:19 min     | $0.10     |
| Structural variants | SV caller                   | omics.c.32xlarge | 31 s         | $0.06     |
| **Total**           |                             |                  | **4:37 min** | **$0.26** |
| Old pipeline        | pbmm2, DeepVariant, sawfish | omics.c.32xlarge | 55:31 min    | $6.80     |

The old pipeline: pbmm2 v26.2.99, DeepVariant 1.10.0 (CPU) and sawfish 2.2.1 on the same genome. Instances are emulated: [Method](docs/method.md).

## Accuracy at a glance

F1 of the whole pipeline. New: pbweave, then pbdv and the SV caller. Old: pbmm2, then DeepVariant and sawfish. Best in bold. Recall, precision, FN, FP and Parabricks: [Accuracy](docs/accuracy.md).

**60 Gb**

| Variants |     New     |     Old     |
| -------- | ----------- | ----------- |
| SNP      | 99.865%     | **99.866%** |
| INDEL    | **98.749%** | 98.738%     |
| SV       | **98.32%**  | 97.67%      |

**90 Gb**

| Variants |     New     |     Old     |
| -------- | ----------- | ----------- |
| SNP      | **99.924%** | **99.924%** |
| INDEL    | **99.567%** | 99.561%     |
| SV       | **98.57%**  | 97.89%      |

## More

- Tools: [pbweave](docs/pbweave.md), [pbdv](docs/pbdv.md), [SV caller](docs/sv-caller.md), [pbhappy](docs/pbhappy.md), [pbtruvari](docs/pbtruvari.md)
- [Accuracy](docs/accuracy.md): all small-variant and SV results
- [Reproduce](docs/reproduce.md): every file, version and command, and the checksums to compare with
- [Method](docs/method.md): hardware, instances, runs

## Downloads

Release [20261021](https://github.com/PacificBiosciences/pblab/releases/tag/20261021): pbweave, pbdv, pbhappy, pbtruvari and pbsamoa 26.3.99 for Linux x86-64 (glibc 2.17 or newer; pbdv on a GPU needs an NVIDIA driver). The SV caller is a Research Experiment and not part of the release.

|                 File                 |                              SHA-256                               |
| ------------------------------------ | ------------------------------------------------------------------ |
| `pblab-20261021-linux-x86_64.tar.xz` | `46cd78f3759a35be9c05d00035244cccda5300ea9df9f5f5ca749d179e9f0737` |

Inside: `bin/`, `SHA256SUMS` of the binaries, `LICENSE`. License: BSD-3-Clause-Clear, [LICENSE](LICENSE).
