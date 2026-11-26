# pbtruvari

SV benchmarking with bench, refine and ga4gh: a reimplementation of truvari 4.2.2 in C++, written after reading its source. It gives the same records as truvari 4.2.2.

All numbers: HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

| Reads | Comparison                                        | Setup                       | Time     | Peak memory |
| ----- | ------------------------------------------------- | --------------------------- | -------- | ----------- |
| 60 Gb | pbcall VCF vs T2T-Q100 v1.1: bench, refine, ga4gh | 128 cores + SMT, 64 threads | **24 s** | 7 GiB       |
| 90 Gb | pbcall VCF vs T2T-Q100 v1.1: bench, refine, ga4gh | 128 cores + SMT, 64 threads | **24 s** | 7 GiB       |


```bash
REF=human_GRCh38_no_alt_analysis_set.fasta
bcftools norm -m-any -Oz --write-index=tbi -o split.vcf.gz hg002.vcf.gz   # multi-allelic records split
pbtruvari bench --reference $REF --includebed GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz \
    --base GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz --comp split.vcf.gz --output tv --passonly --pick ac --dup-to-ins -j 64
pbtruvari refine --reference $REF --regions tv/candidate.refine.bed --recount --use-region-coords \
    --use-original-vcfs --threads 64 --align mafft tv
pbtruvari ga4gh --input tv --output tv/ga4gh_with_refine --with-refine -j 64
```

Download: release [20261126b](../README.md#downloads).

[Back to the overview](../README.md)
