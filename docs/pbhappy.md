# pbhappy

Small-variant benchmarking against GIAB with stratifications: a reimplementation of hap.py and its RTG Tools vcfeval engine in C++, written after reading their source.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads | Comparison                                              | Setup                            | Time      | Peak memory |
| ----- | ------------------------------------------------------- | -------------------------------- | --------- | ----------- |
| 60 Gb | pbcall VCF vs T2T-Q100 v1.1, 127 strata, vcfeval engine | 128 cores + SMT, `--threads 256` | **12 s**  | 26 GiB      |
| 90 Gb | pbcall VCF vs T2T-Q100 v1.1, 127 strata, vcfeval engine | 128 cores + SMT, `--threads 256` | **15 s**  | 26 GiB      |
| 60 Gb | pbcall VCF vs GIAB v4.2.1, 127 strata, vcfeval engine   | 128 cores + SMT, `--threads 256` | **7.4 s** | 25 GiB      |
| 90 Gb | pbcall VCF vs GIAB v4.2.1, 127 strata, vcfeval engine   | 128 cores + SMT, `--threads 256` | **8.4 s** | 26 GiB      |

- It scored the small variants of these pages: [Accuracy](accuracy.md). `q.vcf.gz`: the pbcall VCF with its haploid
  chrX and chrY calls written as homozygous ([Reproduce](reproduce.md#6-scoring)).

```bash
pbhappy HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz q.vcf.gz --pass-only -r human_GRCh38_no_alt_analysis_set.fasta \
    -f HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz \
    --stratification v2.0-GRCh38-stratifications.tsv --engine=vcfeval --threads 256 -o v421
```

Download: release [20261126](../README.md#downloads).

[Back to the overview](../README.md)
