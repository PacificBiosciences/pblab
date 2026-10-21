# pbweave

HiFi mapper and aligner. Writes BAM or CRAM, unsorted or sorted with index.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads |      Output      |                   Setup                    |   Time   | Peak memory |
| ----- | ---------------- | ------------------------------------------ | -------- | ----------- |
| 60 Gb | unsorted BAM     | 128 cores + SMT, `-j 256`                  | **21 s** | 10 GB       |
| 60 Gb | unsorted BAM     | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **24 s** | 7 GB        |
| 60 Gb | sorted BAM + BAI | 128 cores + SMT, `-j 256`                  | **29 s** | 58 GB       |
| 60 Gb | sorted BAM + BAI | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **30 s** | 61 GB       |
| 90 Gb | unsorted BAM     | 128 cores + SMT, `-j 256`                  | **31 s** | 10 GB       |
| 90 Gb | unsorted BAM     | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **34 s** | 7 GB        |
| 90 Gb | sorted BAM + BAI | 128 cores + SMT, `-j 256`                  | **47 s** | 73 GB       |
| 90 Gb | sorted BAM + BAI | omics.c.32xlarge: 64 cores + SMT, `-j 128` | **47 s** | 73 GB       |

- The sort keeps its spill files in memory up to 128 GiB by default; set `--max-memory` for less.
- The sorted BAM is the input of pbdv and the SV caller.
- For reference: pbmm2 v26.2.99 (`--preset HIFI --sort`) takes 15:01 min (60 Gb) and 22:31 min (90 Gb) on omics.c.32xlarge.

```bash
pbweave index -j 128 human_GRCh38_no_alt_analysis_set.fasta GRCh38.wvi   # once
pbweave align -j 128 --sort GRCh38.wvi reads.bam aligned.bam               # without --sort: unsorted
```

Commands for this data set: [Reproduce](reproduce.md).

[Back to the overview](../README.md)
