# pbhappy

Small-variant benchmarking against GIAB with stratifications: a reimplementation of hap.py and its RTG Tools vcfeval engine in C++, written after reading their source.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads |                      Comparison                     |              Setup               |    Time   | Peak memory |
| ----- | --------------------------------------------------- | -------------------------------- | --------- | ----------- |
| 60 Gb | pbdv VCF vs GIAB v4.2.1, 127 strata, vcfeval engine | 128 cores + SMT, `--threads 256` | **2.2 s** | 24 GB       |
| 90 Gb | pbdv VCF vs GIAB v4.2.1, 127 strata, vcfeval engine | 128 cores + SMT, `--threads 256` | **2.2 s** | 24 GB       |

- It scored the small variants of these pages: [Accuracy](accuracy.md).

```bash
pbhappy HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz small.vcf --pass-only -r human_GRCh38_no_alt_analysis_set.fasta \
    -f HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz \
    --stratification v2.0-GRCh38-stratifications.tsv --engine=vcfeval --threads 64 -o small
```

[Back to the overview](../README.md)
