# Reproduce

Every number of these pages comes from public data and the commands below. Check each download with its
checksum: the comparisons depend on exact files.

## 1. Data

|                File               |                                                                                                                    Where                                                                                                                    |                                        Checksum                                        |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| HiFi reads                        | `https://downloads.pacbcloud.com/public/2026Q2/HG002-SPRQ-Nx/Use1/hifi_reads/m84031_260504_231011_s1.hifi_reads.bc2002.bam`                                                                                                                 | md5 `0a4a60c45f1409872a6351a14354ca75` (8,907,432 reads, 144,621,006,554 bases)        |
| Read index                        | same URL + `.pbi`                                                                                                                                                                                                                           | md5 `f901a5b5d5e12ddc72c0d4ccc1db4bea`                                                 |
| GRCh38                            | `https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz`, then step 2                                                    | md5 `30f1a99359a5993edbf8c8bc885825a7` (after step 2)                                  |
| GIAB small variants               | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/NISTv4.2.1/GRCh38/`: `HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz` (+ `.tbi`), `HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed`     | VCF md5 `dc750b3807d4af1f7ffec852e9c2f771`; BED md5 `97265e922a97c69a0391cf3f92a89b8b` |
| GIAB stratifications v2.0         | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/genome-stratifications/v2.0/GRCh38/`: `v2.0-GRCh38-stratifications.tsv` and the 127 BEDs it lists, in their subdirectories                                                | TSV md5 `d698d9333c74207ac141ff272679d402`                                             |
| SV truth                          | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/AshkenazimTrio/analysis/NIST_HG002_DraftBenchmark_defrabbV0.019-20241113/`: `GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz` (+ `.tbi`), `GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed` | VCF md5 `bdaf852698e58dcedcd4b6552d7ab47a`; BED md5 `2ea7eb62a5c7a31d7d4163d8966d07f2` |
| Tandem-repeat catalog (SV caller) | `https://zenodo.org/records/8329210/files/adotto_repeats.hg38.bed.gz`                                                                                                                                                                       | md5 `9cfe0da4028cf7f7502162ee090199e4`                                                 |
| sawfish inputs                    | `https://raw.githubusercontent.com/PacificBiosciences/sawfish/v2.2.1/data/expected_cn/expected_cn.hg38.XY.bed`, `.../data/cnv_excluded_regions/annotation_and_common_cnv.hg38.bed.gz` (+ `.tbi`)                                            | md5 `e7bf2861de660e74a27232b5a2136d9a`, `5b1e65ed94fa62a80240df7c676c6463`             |

Use the SV truth of draft V0.019: the later draft V0.020 has the same file names and other content.

## 2. Prepare the data

```bash
# GRCh38: the NCBI file has 94 IUPAC ambiguity codes (R, Y, K, M, S, W, B); set them to N
zcat GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz | sed '/^>/!s/[RYKMSWBDHVrykmswbdhv]/N/g' \
    > human_GRCh38_no_alt_analysis_set.fasta
samtools faidx human_GRCh38_no_alt_analysis_set.fasta
# BEDs: bgzip and index
bgzip HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed
tabix -p bed HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz
bgzip GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed
tabix -p bed GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz
# tandem-repeat catalog: sort by position, BGZF
zcat adotto_repeats.hg38.bed.gz \
    | LC_ALL=C sort -k1,1V -k2,2n \
    | bgzip > adotto_repeats.hg38.sorted.bed.gz
# content check, md5 59e9573958b00d1566add5f9d6ebdd78:
zcat adotto_repeats.hg38.sorted.bed.gz \
    | md5sum
```

## 3. Software

|                    Tool                    |   Version    |                                                                                                 Where                                                                                                  |
| ------------------------------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| pbweave, pbdv, pbhappy, pbtruvari, pbsamoa | 26.3.99      | release [20261021](https://github.com/PacificBiosciences/pblab/releases/tag/20261021): `pblab-20261021-linux-x86_64.tar.xz`, sha256 `46cd78f3759a35be9c05d00035244cccda5300ea9df9f5f5ca749d179e9f0737` |
| SV caller                                  | 26.3.99      | Research Experiment, not released                                                                                                                                                                      |
| pbmm2                                      | v26.2.99     | `https://github.com/PacificBiosciences/pbmm2/releases/download/v26.2.99/pbmm2`, sha256 `07a0da936c5948a042ed6f6c34cf285e6ec71dcf5fa1a021c2ed82d158ace240`                                              |
| DeepVariant                                | 1.10.0 (CPU) | `docker.io/google/deepvariant:1.10.0` (`sha256:4b3c05fcd0683b4cc62626c7f34442eb5aba474871342d5f6a2c1c59683aa769`)                                                                                      |
| Parabricks                                 | 4.7.1-1      | `nvcr.io/nvidia/clara/clara-parabricks:4.7.1-1`                                                                                                                                                        |
| sawfish                                    | 2.2.1        | `quay.io/pacbio/sawfish:2.2.1_build2` (`sha256:5cf8f02790ecb89e652885c57f103fc9597c41f75ee71be2c05510cbbdf68f59`)                                                                                      |
| samtools, bgzip, tabix                     | any recent   | for the data steps and checks                                                                                                                                                                          |

## 4. Read sets

pbsamoa keeps random ZMWs until their bases reach the target; the seed fixes the choice. The `.pbi` must be
next to the BAM.

```bash
for y in 60G 90G; do
    pbsamoa subsample --yield $y --seed 2405 --threads 32 m84031_260504_231011_s1.hifi_reads.bc2002.bam hg002.$y.reads.bam
done
```

## 5. New pipeline

```bash
REF=human_GRCh38_no_alt_analysis_set.fasta
pbweave index -j 128 $REF GRCh38.wvi                           # once
pbweave align -j 128 --sort GRCh38.wvi hg002.90G.reads.bam aligned.bam   # without --sort: unsorted BAM
pbdv run --device gpu --gpu-id 0 -j 128 --ref $REF --output-vcf small.vcf --output-gvcf small.g.vcf aligned.bam
# SV caller: Research Experiment, not released; the command that made the numbers of these pages
svcaller run -j 128 --haploid-xy --tr-catalog adotto_repeats.hg38.sorted.bed.gz $REF aligned.bam sv.vcf.gz
```

Thread counts as in [Method](method.md); on the L4, pbdv ran with `-j 16`.

## 6. Old pipeline

```bash
pbmm2 index --preset HIFI -j 64 $REF GRCh38.hifi.mmi           # once
pbmm2 align --preset HIFI -j 128 --sort GRCh38.hifi.mmi hg002.90G.reads.bam pbmm2.bam
docker run -v $PWD:$PWD -w $PWD google/deepvariant:1.10.0 /opt/deepvariant/bin/run_deepvariant \
    --model_type=PACBIO --ref=$PWD/$REF --reads=$PWD/pbmm2.bam --output_vcf=$PWD/dv.vcf.gz --num_shards=128 \
    --intermediate_results_dir=$PWD/tmp
docker run --gpus device=0 -v $PWD:$PWD -w $PWD nvcr.io/nvidia/clara/clara-parabricks:4.7.1-1 pbrun deepvariant \
    --mode pacbio --num-gpus 1 --ref $PWD/$REF --in-bam $PWD/pbmm2.bam --out-variants $PWD/parabricks.vcf --tmp-dir $PWD/tmp
docker run -v $PWD:$PWD -w $PWD quay.io/pacbio/sawfish:2.2.1_build2 bash -c "set -eo pipefail
    sawfish discover --threads 64 --ref $REF --bam pbmm2.bam --expected-cn expected_cn.hg38.XY.bed \
        --cnv-excluded-regions annotation_and_common_cnv.hg38.bed.gz --output-dir discover
    sawfish joint-call --threads 64 --sample discover --output-dir call
    sawshark --threads 64 --vcf call/genotyped.sv.vcf.gz \
        | bcftools view -Oz --write-index=tbi -o sawfish.vcf.gz -"
```

## 7. Scoring

```bash
# small variants (same for small.vcf, dv.vcf.gz, parabricks.vcf) -> small.summary.csv, rows SNP/INDEL PASS
pbhappy HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz small.vcf --pass-only -r $REF \
    -f HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz \
    --stratification v2.0-GRCh38-stratifications.tsv --engine=vcfeval --threads 64 -o small

# SVs (same for sv.vcf.gz and sawfish.vcf.gz) -> tv/refine.variant_summary.json; truvari 4.2.2 gives the same records
pbtruvari bench --reference $REF --includebed GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz \
    --base GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz --comp sv.vcf.gz --output tv --passonly --pick ac --dup-to-ins -j 64
pbtruvari refine --reference $REF --regions tv/candidate.refine.bed --recount --use-region-coords \
    --use-original-vcfs --threads 64 --align mafft tv
```

## Check your results

|              Check               |               60 Gb                |               90 Gb                |
| -------------------------------- | ---------------------------------- | ---------------------------------- |
| Read set records (command below) | `30934a7f089c6b681d9b2c1fd1a98f46` | `0cc039c6e73009c112b3436b90b98ad4` |
| pbdv VCF md5, RTX PRO 6000       | `523f8db15209bdd37e6ec08c747687d0` | `e0bbdbe4bd845907723131d5f1d0417e` |
| pbdv gVCF md5, RTX PRO 6000      | `77776898bebf0e3d8ab1f7cec6a3182f` | `a751be872d64e80061f35b76c83eb797` |
| pbdv VCF md5, L4                 | `584eeabd8dd9acdb2ff31d9b04d674d4` | `5f697dd5c7a540b8c7636619a593cedd` |
| pbdv gVCF md5, L4                | `6bd5bc242b975c646a5e377919d9ca93` | `5e632effd2550080e131abb04de666b4` |

```bash
samtools view hg002.60G.reads.bam \
    | md5sum   # records only, without the header
```

The read-set file md5 depends on the `@PG` command line in the header, so compare records. The pbdv md5s
are of the whole files (their headers hold no paths). 16 and 128 threads give the same pbdv output; other GPU
models can differ in the last bits.

[Back to the overview](../README.md)
