# pbcall

> **Experimental**: pbcall is not in the release yet. These pages show what it does today.

Variant caller for PacBio HiFi: SNVs, indels and structural variants of one sample in one VCF, phased. BAM in
(sorted, with index), VCF out.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38; the input is the
sorted [pbalign](pbalign.md) BAM, `--haploid-xy` (HG002 is male), with the haplotype panel of the release (221
samples; HG002 and its parents are not in it).

## Fastest

One AMD EPYC 9755 CPU, 128 cores + SMT, `-j 256`; with `--device gpu`, one RTX PRO 6000.

| Reads | Device | pbcall time  | Peak memory | Records   |
| ----- | ------ | ------------ | ----------- | --------- |
| 60 Gb | GPU    | **48 s**     | 51 GiB      | 6,808,858 |
| 60 Gb | CPU    | **1:11 min** | 46 GiB      | 6,808,858 |
| 90 Gb | GPU    | **1:03 min** | 67 GiB      | 6,929,091 |
| 90 Gb | CPU    | **1:40 min** | 57 GiB      | 6,929,091 |

## Cheapest

AWS EC2 m7a.16xlarge spot instance (64 cores, 256 GiB, `--device cpu -j 64`), us-east-1c, 2026-11-25, at the spot
price of that day ($0.7546/h). Compute cost: the pbcall run itself at the spot price of the instance.

| Reads | pbcall time  | Compute cost | Peak memory |
| ----- | ------------ | ------------ | ----------- |
| 60 Gb | **3:19 min** | **$0.042**   | 27 GiB      |
| 90 Gb | **4:49 min** | **$0.061**   | 36 GiB      |

The whole instance, from launch to shutdown (start, fetch of the BAM, reference, pbcall and panel from S3, pbcall,
upload of the VCF to S3): 60 Gb 3:51 min ($0.049), 90 Gb 5:28 min ($0.069).

## Same calls everywhere

GPU and CPU, 256 and 64 threads give the same records (`zcat hg002.vcf.gz | grep -v '^#' | md5sum`): 60 Gb
`c76c60bab1ef4d28bab1ceec9b1b8eb0`, 90 Gb `8d5be80e755dcf63d53115d46a068761`. Only the header lines with the command
line and the panel path differ.

- Phasing (60 Gb): 2,989,228 of the 3,017,138 heterozygous PASS records are phased
  (`bcftools view -H -f PASS -g het hg002.vcf.gz`, a `|` in GT).
- Accuracy: [Accuracy](accuracy.md) (workflows A and B).

## With the assembly and the lift

As in the [Lift](pbalign.md#lift): pbcall reads the lifted BAM and, with `--asm-bam`, the contigs of both haplotypes on
GRCh38 (the sorted contig BAMs); it then uses its model for calls with an assembly.

### Fastest

| Reads | Device | pbcall time  | Peak memory | Records   |
| ----- | ------ | ------------ | ----------- | --------- |
| 60 Gb | GPU    | **59 s**     | 61 GiB      | 6,833,592 |
| 60 Gb | CPU    | **1:21 min** | 57 GiB      | 6,833,592 |
| 90 Gb | GPU    | **1:15 min** | 78 GiB      | 6,957,817 |
| 90 Gb | CPU    | **1:52 min** | 68 GiB      | 6,957,817 |

### Cheapest

AWS EC2 m7a.16xlarge spot instance (`--device cpu -j 64`), 2026-11-25, at the spot price of that day: 60 Gb in
us-east-1d ($0.9335/h), 90 Gb in us-east-1c ($0.7546/h). The whole chain after the assembly on one instance, one step
after the other. Compute cost in bold.

| Step                    | 60 Gb                         | 90 Gb                          |
| ----------------------- | ----------------------------- | ------------------------------ |
| Contigs hap1 on GRCh38  | 1:11 min (**$0.018**), 66 GiB | 3:36 min (**$0.045**), 119 GiB |
| Contigs hap2 on GRCh38  | 1:39 min (**$0.026**), 69 GiB | 1:58 min (**$0.025**), 113 GiB |
| Index of hap1 + hap2    | 11 s (**$0.003**), 20 GiB     | 13 s (**$0.003**), 20 GiB      |
| Lift, sorted BAM        | 2:28 min (**$0.038**), 61 GiB | 3:38 min (**$0.046**), 64 GiB  |
| pbcall with `--asm-bam` | 3:39 min (**$0.057**), 35 GiB | 5:12 min (**$0.065**), 45 GiB  |
| **Whole instance**      | **9:58 min ($0.155)**         | **15:32 min ($0.195)**         |

The records are the same on the GPU and the CPU (Fastest), and in the cloud with 64 threads: 60 Gb
`b75ddb6295bf901fe9d5f09ac7a08a37`, 90 Gb `ca7192202000ccb29931479a78dd1dff`.

```bash
pbcall call -j 64 --haploid-xy --panel pbcall.panel --asm-bam hap1.bam,hap2.bam \
    human_GRCh38_no_alt_analysis_set.fasta lifted.bam hg002.vcf.gz
```

```bash
pbcall call -j 64 --haploid-xy --panel pbcall.panel human_GRCh38_no_alt_analysis_set.fasta aligned.bam hg002.vcf.gz
```

Without `--panel`, pbcall looks for the panel at `share/pbcall/pbcall.panel` next to its binary. `--device cpu`
or `gpu` choose where the read likelihoods run; both give the same VCF.

[Back to the overview](../README.md)
