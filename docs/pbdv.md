# pbdv

A reimplementation of DeepVariant 1.10.0 (PACBIO model) in C++ and CUDA, written after reading the DeepVariant source. BAM in, VCF and gVCF out.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads |                         Setup                          |     Time     | Peak memory |
| ----- | ------------------------------------------------------ | ------------ | ----------- |
| 60 Gb | 128 cores + SMT, 1 RTX PRO 6000, `-j 128`              | **30 s**     | 64 GB       |
| 60 Gb | omics.g6.4xlarge: 16 vCPU, 1 L4, 60 GiB limit, `-j 16` | **2:45 min** | 33 GB       |
| 90 Gb | 128 cores + SMT, 1 RTX PRO 6000, `-j 128`              | **33 s**     | 78 GB       |
| 90 Gb | omics.g6.4xlarge: 16 vCPU, 1 L4, 60 GiB limit, `-j 16` | **3:19 min** | 41 GB       |

- Input: the sorted pbweave BAM. Accuracy: [Accuracy](accuracy.md) (the RTX PRO 6000 calls).
- 16 and 128 threads give the same VCF. GPU models can differ in the last bits of floating point, so the VCF of an L4 is not byte-identical to the VCF of an RTX PRO 6000: md5s of both in [Reproduce](reproduce.md#check-your-results).
- For reference, on the pbmm2 BAM: DeepVariant 1.10.0 takes 23:35 min (60 Gb) and 25:52 min (90 Gb) on omics.c.32xlarge (no GPU); Parabricks 4.7.1 takes 6:53 min and 8:00 min with the RTX PRO 6000 (VCF only).

```bash
pbdv run --device gpu --gpu-id 0 -j 128 --ref human_GRCh38_no_alt_analysis_set.fasta \
    --output-vcf small.vcf --output-gvcf small.g.vcf aligned.bam
```

[Back to the overview](../README.md)
