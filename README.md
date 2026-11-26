# pblab: PacBio HiFi analysis tools

```text
  .-----------------.         _     _       _
  | o o o o o o o o |   _ __ | |__ | | __ _| |__
  | o o o * o o o o |  | '_ \| '_ \| |/ _` | '_ \
  | o o o o o o * o |  | |_) | |_) | | (_| | |_) |
  | o * o o o o o o |  | .__/|_.__/|_|\__,_|_.__/
  '-----------------'  |_|
     |                   Minutes, not hours.
     '--> HiFi => assemble => align => call => score
```

Fast tools for HiFi data analysis, tuned to work together: pbalign aligns, pbhifiasm assembles, pbdv calls small variants, pbhappy and pbtruvari score calls against truth sets; pbcall (experimental, not part of release 20261126) calls SNVs, indels and SVs. [Download](#downloads) release 20261126. The human numbers are for HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38; [Accuracy](docs/accuracy.md) adds HG003 as the held-out genome, and [pbalign](docs/pbalign.md#other-genomes) seven more species.

pbhifiasm, pbdv, pbhappy and pbtruvari reimplement hifiasm, DeepVariant, hap.py (with RTG Tools vcfeval) and truvari. pblab is not affiliated with or endorsed by Google, Illumina, Real Time Genomics or the authors of these tools; their names describe what is reimplemented.

## Fastest

Time per genome on one AMD EPYC 9755 CPU, 128 cores + SMT (256 vCPU); pbcall and pbdv with one RTX PRO 6000.

| Tool                           | Task                        | Status       | 60 Gb        | 90 Gb        |
| ------------------------------ | --------------------------- | ------------ | ------------ | ------------ |
| [pbalign](docs/pbalign.md)     | Alignment, unsorted BAM     | preview      | **21 s**     | **32 s**     |
|                                | Alignment, sorted BAM + BAI |              | **27 s**     | **40 s**     |
| [pbhifiasm](docs/pbhifiasm.md) | Assembly, both haplotypes   | preview      | **2:12 min** | **4:18 min** |
| [pbcall](docs/pbcall.md)       | SNV, indel and SV calling   | experimental | **48 s**     | **1:03 min** |
| [pbdv](docs/pbdv.md)           | Small-variant calling       | preview      | **29 s**     | **33 s**     |
| [pbhappy](docs/pbhappy.md)     | Small-variant benchmarking  | preview      | **12 s**     | **15 s**     |
| [pbtruvari](docs/pbtruvari.md) | SV benchmarking             | preview      | **24 s**     | **24 s**     |

## Cheapest

A genome in the cloud: reads to sorted BAM, then the variants, one step after the other on AWS EC2 spot instances, us-east-1, 2026-11-25. Compute cost: the run of the tool itself at the spot price of the day; the whole instance (start, fetch from S3, upload to S3) is on the tool pages. The last three rows are the assembly route (workflow B of [Accuracy](docs/accuracy.md)): the assembly from the reads, its contigs on GRCh38 and the lift of the reads (one pbalign step, with the contig index), then pbcall with the lifted reads and the contigs; it needs no separate alignment. The steps after the assembly ran on one instance.

**60 Gb**

| Step                                | Tool      | Instance          | Time     | Compute cost |
| ----------------------------------- | --------- | ----------------- | -------- | ------------ |
| Align + sort                        | pbalign   | m7a.16xlarge      | 59 s     | $0.015       |
| SNVs, indels, SVs                   | pbcall    | m7a.16xlarge      | 3:19 min | $0.042       |
| Small variants                      | pbdv      | g6.2xlarge (1 L4) | 5:42 min | $0.077       |
| Assembly                            | pbhifiasm | m7a.16xlarge      | 6:14 min | $0.102       |
| Contigs on GRCh38 + lift            | pbalign   | m7a.16xlarge      | 5:29 min | $0.085       |
| SNVs, indels, SVs with the assembly | pbcall    | m7a.16xlarge      | 3:39 min | $0.057       |

**90 Gb**

| Step                                | Tool      | Instance          | Time      | Compute cost |
| ----------------------------------- | --------- | ----------------- | --------- | ------------ |
| Align + sort                        | pbalign   | m7a.16xlarge      | 1:28 min  | $0.023       |
| SNVs, indels, SVs                   | pbcall    | m7a.16xlarge      | 4:49 min  | $0.061       |
| Small variants                      | pbdv      | g6.2xlarge (1 L4) | 7:00 min  | $0.096       |
| Assembly                            | pbhifiasm | m7a.16xlarge      | 12:24 min | $0.204       |
| Contigs on GRCh38 + lift            | pbalign   | m7a.16xlarge      | 9:25 min  | $0.118       |
| SNVs, indels, SVs with the assembly | pbcall    | m7a.16xlarge      | 5:12 min  | $0.065       |


**Per workflow** (the sum of its steps above; workflows as in [Accuracy](docs/accuracy.md))

| Workflow                               | 60 Gb                 | 90 Gb                 |
| -------------------------------------- | --------------------- | --------------------- |
| A: pbalign → pbcall                    | 4:18 min, **$0.057**  | 6:17 min, **$0.084**  |
| B: pbhifiasm → contigs + lift → pbcall | 15:22 min, **$0.244** | 27:01 min, **$0.387** |
| C: pbalign → pbdv                      | 6:41 min, **$0.092**  | 8:28 min, **$0.119**  |

[Method](docs/method.md).

## Accuracy at a glance

F1 against the T2T-Q100 v1.1 truth of HG002 (small variants: pbhappy; SVs: pbtruvari). A: pbalign, then pbcall. B: pbhifiasm,
the contigs on GRCh38, the lift, then pbcall with the assembly. C: pbalign, then pbdv. GIAB v4.2.1, aardvark, recall,
precision, FN and FP: [Accuracy](docs/accuracy.md).

**60 Gb**

| Variants | A: pbcall | B: assembly + lift + pbcall | C: pbdv |
| -------- | --------- | --------------------------- | ------- |
| SNP      | 99.720%   | **99.835%**                 | 99.615% |
| INDEL    | 97.807%   | **97.964%**                 | 95.548% |
| SV       | 99.25%    | **99.57%**                  | –       |

**90 Gb**

| Variants | A: pbcall | B: assembly + lift + pbcall | C: pbdv |
| -------- | --------- | --------------------------- | ------- |
| SNP      | 99.797%   | **99.900%**                 | 99.717% |
| INDEL    | 98.744%   | **98.864%**                 | 97.580% |
| SV       | 99.48%    | **99.70%**                  | –       |

## More

- Tools: [pbalign](docs/pbalign.md), [pbhifiasm](docs/pbhifiasm.md), [pbcall](docs/pbcall.md), [pbdv](docs/pbdv.md), [pbhappy](docs/pbhappy.md), [pbtruvari](docs/pbtruvari.md)
- [Accuracy](docs/accuracy.md): all small-variant and SV results
- [Reproduce](docs/reproduce.md): every file, version and command, and the checksums to compare with
- [Method](docs/method.md): hardware, instances, runs
- [Downloads](#downloads): release 20261126

## Downloads

Release [20261126](https://github.com/PacificBiosciences/pblab/releases/tag/20261126): pbalign, pbhifiasm, pbdv, pbhappy, pbtruvari and pbsamoa 26.3.99 for Linux x86-64, the builds that made the numbers of these pages ([Method](docs/method.md#versions)). pbcall is experimental and not part of it. glibc 2.17 or newer; pbdv on a GPU needs an NVIDIA driver.

```bash
curl -LO https://github.com/PacificBiosciences/pblab/releases/download/20261126/pblab-20261126-linux-x86_64.tar.xz
echo "92cee909152146f88acf3c8bf229c8f4935e409d59313d11e350108441c3e3a4  pblab-20261126-linux-x86_64.tar.xz" | sha256sum -c
tar -xJf pblab-20261126-linux-x86_64.tar.xz
export PATH=$PWD/pblab-20261126-linux-x86_64/bin:$PATH
pbalign --version
```

| File                                 | SHA-256                                                            |
| ------------------------------------ | ------------------------------------------------------------------ |
| `pblab-20261126-linux-x86_64.tar.xz` | `92cee909152146f88acf3c8bf229c8f4935e409d59313d11e350108441c3e3a4` |

Inside: `bin/`, `SHA256SUMS` of the binaries and `README.md` with the commits.

Earlier: release [20261021](https://github.com/PacificBiosciences/pblab/releases/tag/20261021) (pbweave, pbdv, pbhappy, pbtruvari, pbsamoa; BSD-3-Clause-Clear), older builds than those of these pages.

---

For Research Use Only. Not for use in diagnostic procedures.
