# SV caller (Research Experiment)

Structural-variant caller for HiFi, tuned together with pbweave. A Research Experiment: not released, so its numbers cannot be reproduced outside PacBio yet.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads |                   Setup                    |   Time   |
| ----- | ------------------------------------------ | -------- |
| 60 Gb | 128 cores + SMT, `-j 256`                  | **19 s** |
| 60 Gb | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **23 s** |
| 90 Gb | 128 cores + SMT, `-j 256`                  | **26 s** |
| 90 Gb | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **31 s** |

- Input: the sorted pbweave BAM, the adotto tandem-repeat catalog. Accuracy: [Accuracy](accuracy.md).
- For reference: sawfish 2.2.1 (discover, joint-call, sawshark) takes 5:40 min (60 Gb) and 7:07 min (90 Gb) on omics.c.32xlarge, on the pbmm2 BAM.

[Back to the overview](../README.md)
