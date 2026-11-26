# pbdv

A reimplementation of DeepVariant 1.10.0 (PACBIO model) in C++ and CUDA, written after reading the DeepVariant source. BAM in, VCF and gVCF out.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38; the input is the sorted
pbalign BAM.

## Fastest

One AMD EPYC 9755 CPU, 128 cores + SMT, one RTX PRO 6000, `-j 128`.

| Reads | pbdv time | Peak memory |
| ----- | --------- | ----------- |
| 60 Gb | **29 s**  | 81 GiB      |
| 90 Gb | **33 s**  | 93 GiB      |

## Cheapest

AWS EC2 g6.2xlarge spot instance (8 vCPU, 32 GiB, 1 NVIDIA L4, `-j 8`), 2026-11-25, at the spot price of that day:
60 Gb in us-east-1a ($0.8141/h), 90 Gb in us-east-1c ($0.8184/h).
Compute cost: the pbdv run itself at the spot price of the instance.

| Reads | pbdv time    | Compute cost | Peak memory |
| ----- | ------------ | ------------ | ----------- |
| 60 Gb | **5:42 min** | **$0.077**   | 25 GiB      |
| 90 Gb | **7:00 min** | **$0.096**   | 23 GiB      |

The whole instance, from launch to shutdown (start, fetch from S3, pbdv, upload of the VCF and gVCF to S3):
60 Gb 9:38 min ($0.131), 90 Gb 11:43 min ($0.160). The 8 vCPU were 90% busy during pbdv.

- Accuracy: [Accuracy](accuracy.md) (workflow C, the VCF of Fastest).
- GPU models can differ in the last bits of floating point, so the VCF of an L4 is not byte-identical to the VCF of an RTX PRO 6000.

```bash
pbdv run --device gpu --gpu-id 0 -j 128 --ref human_GRCh38_no_alt_analysis_set.fasta \
    --output-vcf small.vcf --output-gvcf small.g.vcf aligned.bam
```

Download: release [20261126b](../README.md#downloads).

[Back to the overview](../README.md)
