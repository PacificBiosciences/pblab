# Method

## Hardware

- AMD EPYC 9755 server: 2 NUMA nodes of 128 cores each, SMT, local NVMe, NVIDIA RTX PRO 6000 Blackwell Server
  Edition GPUs.
- AWS EC2, us-east-1: m7a.16xlarge (AMD EPYC 9R14, 64 cores, 256 GiB) for the CPU steps, g6.2xlarge (AMD EPYC 7R13,
  8 vCPU, 32 GiB, 1 NVIDIA L4) for pbdv.

## Setups

- **Fastest** (the README and the tool pages): one AMD EPYC 9755 CPU of the server (one NUMA node), 128 cores + SMT
  (256 vCPU), no memory limit; one run per CPU at a time. Threads: `-j 256` for pbalign, pbhifiasm and pbcall,
  `-j 128` for pbdv (with one RTX PRO 6000), `--threads 256` for pbhappy, 64 for pbtruvari. The inputs were read into
  memory before the runs.
- **Cheapest** (the README and the tool pages): real EC2 spot instances. The instance starts, fetches its inputs and
  the tool from S3 (AWS CLI with its CRT transfer client), runs the tool with one thread per vCPU, uploads the output
  to S3 and shuts down; inputs, outputs and scratch in memory (m7a.16xlarge) or on local disk (g6.2xlarge). pbalign,
  pbhifiasm, pbcall and pbdv each ran on their own instances; the steps of the assembly route after the assembly
  (contigs on GRCh38, the lift, pbcall with the assembly) ran on one instance, one after the other.

## Runs

- pbalign, HG002 read alignment (Fastest): 1 warmup run, then 3 timed runs per row; the tables show the median.
- pbhifiasm (Fastest): 1 warmup run, then 1 timed run; both gave the same contigs.
- All other Fastest rows (contig alignment, lift, pbcall, pbdv, pbhappy, pbtruvari, the other genomes): one timed run
  each, after the inputs were read into memory.
- Cheapest: each instance starts cold. pbalign: the mean of 3 instances per data set; pbhifiasm, pbcall, pbdv and the
  assembly route: one instance per data set.
- Time is the wall clock of the tool's command; peak memory is max RSS. The tool pages also give the whole instance
  of Cheapest, from launch to shutdown.
- Compute cost = the time of the tool's run x the spot price per hour of the instance on the day of the run
  (us-east-1), from the exact seconds; the tables round the time to whole seconds.
- Accuracy: workflow C is the pbdv VCF of the Fastest run (RTX PRO 6000). The L4 VCF of Cheapest differs from it in
  the last bits of floating point; its scores differ by at most 9 variants (pbhappy, aardvark GT).

## Versions

Measured 2026-11-25 and 2026-11-26. Release [20261126](../README.md#downloads) ships these builds:

| Tool      | Commit  | Note                                                                                                               |
| --------- | ------- | ------------------------------------------------------------------------------------------------------------------ |
| pbalign   | 4e722c5 | the other genomes, Cheapest and the BAMs of pbcall and pbdv ran ed11c12, the version before, with the same records |
| pbhifiasm | 863893b | the CPU build; HG003 ran another CPU build of the same commit                                                      |
| pbdv      | eb4c1ca | Cheapest ran another build of the same commit                                                                      |
| pbhappy   | 92cce05 |                                                                                                                    |
| pbtruvari | 1a4cf4b |                                                                                                                    |
| pbsamoa   | e0b0f08 |                                                                                                                    |
| pbcall    | e3f414c | experimental, not part of the release; with its panel (221 samples, without HG002, HG003, HG004)                   |

All report version 26.3.99. Other tools: bcftools 1.23, aardvark v1.2.0 ([Reproduce](reproduce.md#3-software)).

[Back to the overview](../README.md)
