# Reproduce

Every number of these pages comes from public data and the commands below. Check each download with its
checksum: the comparisons depend on exact files.

## 1. Data

| File                      | Where                                                                                                                                                                                                                                       | Checksum                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| HG002 HiFi reads          | `https://downloads.pacbcloud.com/public/2026Q2/HG002-SPRQ-Nx/Use1/hifi_reads/m84031_260504_231011_s1.hifi_reads.bc2002.bam`                                                                                                                 | md5 `0a4a60c45f1409872a6351a14354ca75` (8,907,432 reads, 144,621,006,554 bases)        |
| HG002 read index          | same URL + `.pbi`                                                                                                                                                                                                                           | md5 `f901a5b5d5e12ddc72c0d4ccc1db4bea`                                                 |
| GRCh38                    | `https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz`, then step 2                                                    | md5 `30f1a99359a5993edbf8c8bc885825a7` (after step 2)                                  |
| HG002 GIAB small variants | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/NISTv4.2.1/GRCh38/`: `HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz` (+ `.tbi`), `HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed`     | VCF md5 `dc750b3807d4af1f7ffec852e9c2f771`; BED md5 `97265e922a97c69a0391cf3f92a89b8b` |
| HG003 HiFi reads          | `https://downloads.pacbcloud.com/public/revio/2024Q4/WGS/GIAB_trio/HG003/m84039_241002_000337_s3.hifi_reads.bc2020.bam`                                                                                                                     | md5 `f53c07efc4d95449ec003fadf3527dd7` (7,790,978 reads, 109,773,789,328 bases)        |
| HG003 read index          | same URL + `.pbi`                                                                                                                                                                                                                           | md5 `bf09ef672e09c1c893f02894c5ef5a8e`                                                 |
| HG003 GIAB small variants | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG003_NA24149_father/NISTv4.2.1/GRCh38/`: `HG003_GRCh38_1_22_v4.2.1_benchmark.vcf.gz` (+ `.tbi`), `HG003_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed`  | VCF md5 `ea98d8b9de762e050e34666462bf3452`; BED md5 `26f7a0a353c653e4e2979654586110b1` |
| GIAB stratifications v2.0 | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/genome-stratifications/v2.0/GRCh38/`: `v2.0-GRCh38-stratifications.tsv` and the 127 BEDs it lists, in their subdirectories                                                | TSV md5 `d698d9333c74207ac141ff272679d402`                                             |
| T2T-Q100 small variants   | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/AshkenazimTrio/analysis/NIST_HG002_DraftBenchmark_defrabbV0.019-20241113/`: `GRCh38_HG2-T2TQ100-V1.1_smvar.vcf.gz` (+ `.tbi`), `GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed` | VCF md5 `a77065ecb99ece71939be763d7811209`; BED md5 `77a7f0b49d8bbbab16d346cf1a086ba4` |
| SV truth                  | `https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/AshkenazimTrio/analysis/NIST_HG002_DraftBenchmark_defrabbV0.019-20241113/`: `GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz` (+ `.tbi`), `GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed` | VCF md5 `bdaf852698e58dcedcd4b6552d7ab47a`; BED md5 `2ea7eb62a5c7a31d7d4163d8966d07f2` |

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
bgzip HG003_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed
tabix -p bed HG003_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz
bgzip GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed
tabix -p bed GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz
bgzip GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed
tabix -p bed GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed.gz
```

## 3. Software

| Tool                                                  | Version           | Where                                                                                                                                                                                                                                               |
| ----------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pbalign, pbhifiasm, pbdv, pbhappy, pbtruvari, pbsamoa | 26.3.99           | release [20261126](https://github.com/PacificBiosciences/pblab/releases/tag/20261126): `pblab-20261126-linux-x86_64.tar.xz`, sha256 `92cee909152146f88acf3c8bf229c8f4935e409d59313d11e350108441c3e3a4`; the commits in [Method](method.md#versions) |
| pbcall                                                | 26.3.99 (e3f414c) | experimental, not part of the release; with its panel `pbcall.panel`                                                                                                                                                                                |
| bcftools                                              | 1.23              | for the scoring steps (`--write-index` needs 1.18 or newer)                                                                                                                                                                                         |
| bedtools                                              | any recent        | for the A-or-B-only check                                                                                                                                                                                                                           |
| aardvark                                              | v1.2.0            | `https://github.com/PacificBiosciences/aardvark/releases/download/v1.2.0/aardvark-v1.2.0-x86_64-unknown-linux-gnu.tar.gz`, sha256 `b6707c8e89214f1d032269ccdf1163eb8a0edd47229696898dfa1ec1aa7c69d0`                                                |
| samtools, bgzip, tabix                                | any recent        | for the data steps and checks                                                                                                                                                                                                                       |

## 4. Read sets

pbsamoa keeps random ZMWs until their bases reach the target. Without `--seed`, it derives the seed from the
header of the input (as samtools 1.24 and later), so the same file always gives the same reads: seed 2664284907
for this BAM. The `.pbi` must be next to the BAM.

```bash
for y in 60G 90G; do
    pbsamoa subsample --yield $y m84031_260504_231011_s1.hifi_reads.bc2002.bam hg002.$y.reads.bam
done
```

| Set | Reads     | Bases          | Records (see [Check your results](#check-your-results)) |
| --- | --------- | -------------- | ------------------------------------------------------- |
| 60G | 3,695,102 | 60,000,011,335 | `13cbe0e62e5d3ad87b59121241bf666b`                      |
| 90G | 5,542,653 | 90,000,006,862 | `0f5f2c935447606145df3158027961fc`                      |

HG003, the same way:

```bash
for y in 60G 90G; do
    pbsamoa subsample --yield $y m84039_241002_000337_s3.hifi_reads.bc2020.bam hg003.$y.reads.bam
done
```

| Set | Reads     | Bases          | Records                            |
| --- | --------- | -------------- | ---------------------------------- |
| 60G | 4,257,643 | 60,000,013,949 | `55805c5ebc7507ccf3e64df9a5cab75e` |
| 90G | 6,387,170 | 90,000,005,879 | `0e61c87330a4534748f159318007f3c8` |

## 5. Pipeline

```bash
For each read set (here hg002.90G; the same for 60G and for HG003):

```bash
REF=human_GRCh38_no_alt_analysis_set.fasta
pbalign index -j 256 $REF GRCh38.pai                                                # once
pbalign index -j 256 --preset CONTIG $REF GRCh38.contig.pai                         # once, for B

# A: pbalign -> pbcall
pbalign align -j 256 --sort GRCh38.pai hg002.90G.reads.bam aligned.bam              # without --sort: unsorted; aligned.cram: CRAM
pbcall call -j 256 --haploid-xy --panel pbcall.panel $REF aligned.bam hg002.vcf.gz  # SNVs, indels, SVs

# B: pbhifiasm -> contigs + lift -> pbcall
pbhifiasm -j 256 -o asm hg002.90G.reads.bam                                         # asm.bp.hap{1,2}.p_ctg.gfa
for h in hap1 hap2; do
    awk -F'\t' '$1 == "S" {print ">"$2; print $3}' asm.bp.$h.p_ctg.gfa > $h.fa
    pbalign align -j 256 --preset CONTIG --sort GRCh38.contig.pai $h.fa $h.bam
done
cat hap1.fa hap2.fa > contigs.fa
pbalign index -j 256 contigs.fa contigs.pai
pbalign align -j 256 --sort --assembly-index contigs.pai --assembly-bam hap1.bam,hap2.bam GRCh38.pai \
    hg002.90G.reads.bam lifted.bam
pbcall call -j 256 --haploid-xy --panel pbcall.panel --asm-bam hap1.bam,hap2.bam $REF lifted.bam hg002.B.vcf.gz

# C: pbalign -> pbdv
pbdv run --device gpu --gpu-id 0 -j 128 --ref $REF --output-vcf small.vcf --output-gvcf small.g.vcf aligned.bam
```

Threads as in [Method](method.md). Cheapest: one thread per vCPU (`-j 64` on m7a.16xlarge, `-j 8` for pbdv on the
L4), pbalign with `--max-memory 64G` for the contigs and the lift, pbcall with `--device cpu`. Score B's VCF as A's.

## 6. Scoring

```bash
# pbcall writes chrX and chrY outside the PARs haploid (--haploid-xy); write them as homozygous (as hap.py expects):
zcat hg002.vcf.gz | awk 'BEGIN{FS=OFS="\t"} /^#/ {print; next}
    ($1=="chrX"||$1=="chrY") {split($9,f,":"); for(i=1;i<=length(f);i++) if(f[i]=="GT"){n=split($10,v,":");
        if(v[i]!~/[\/|]/) v[i]=v[i]"/"v[i]; s=v[1]; for(j=2;j<=n;j++) s=s":"v[j]; $10=s}} {print}' \
    | bgzip > q.vcf.gz && tabix -p vcf q.vcf.gz
STRATA=v2.0-GRCh38-stratifications.tsv

# small variants -> <prefix>.summary.csv, rows SNP/INDEL PASS
pbhappy GRCh38_HG2-T2TQ100-V1.1_smvar.vcf.gz q.vcf.gz --pass-only -r $REF -f GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed.gz \
    --stratification $STRATA --engine=vcfeval --star-as-ref --gender male --threads 64 -o q100
pbhappy HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz q.vcf.gz --pass-only -r $REF \
    -f HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz --stratification $STRATA --engine=vcfeval --threads 64 -o v421

# SVs -> tv/refine.variant_summary.json; truvari 4.2.2 gives the same records
bcftools norm -m-any -Oz --write-index=tbi -o split.vcf.gz hg002.vcf.gz
pbtruvari bench --reference $REF --includebed GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz \
    --base GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz --comp split.vcf.gz --output tv --passonly --pick ac --dup-to-ins -j 64
pbtruvari refine --reference $REF --regions tv/candidate.refine.bed --recount --use-region-coords \
    --use-original-vcfs --threads 64 --align mafft tv

# aardvark: PASS, no symbolic alleles, biallelic -> <dir>/summary.tsv (row GT/BASEPAIR, ALL, ALL, ALL)
bcftools view -f PASS -e 'ALT~"<"' q.vcf.gz | bcftools norm -m -any \
    | bcftools annotate -x INFO/SVTYPE,INFO/SVLEN,INFO/END -Oz --write-index=tbi -o aq.vcf.gz
aardvark compare --preset small -r $REF -t HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    -b HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz -q aq.vcf.gz -o small_v421
aardvark compare --preset small -r $REF -t GRCh38_HG2-T2TQ100-V1.1_smvar.vcf.gz \
    -b GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed.gz -q aq.vcf.gz -o small_q100
aardvark compare --preset large -r $REF -t GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz \
    -b GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed.gz -q aq.vcf.gz -o full

# A or B only (Accuracy, GIAB v4.2.1): PASS SNPs that only one of A and B calls inside the GIAB v4.2.1 regions; of
# those inside the T2T-Q100 regions, how many the T2T-Q100 truth has (same position, REF and ALT)
export LC_ALL=C
snps() { bcftools view -v snps -i 'GT~"1"' -T $2 "${@:3}" $1 | bcftools norm -m -any \
    | bcftools query -f '%CHROM\t%POS\t%REF\t%ALT\n' | sort -u; }
V421BED=HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz; Q100BED=GRCh38_HG2-T2TQ100-V1.1_smvar.benchmark.bed.gz
snps qA.vcf.gz $V421BED -f PASS > A.txt                             # qA, qB: q.vcf.gz of A and of B
snps qB.vcf.gz $V421BED -f PASS > B.txt
snps GRCh38_HG2-T2TQ100-V1.1_smvar.vcf.gz $Q100BED > q100.txt
comm -13 A.txt B.txt > Bonly.txt; comm -23 A.txt B.txt > Aonly.txt
for f in Bonly Aonly; do
    n=$(awk 'BEGIN{OFS="\t"} {print $1, $2-1, $2}' $f.txt | bedtools intersect -u -a - -b $Q100BED | wc -l)
    echo "$f: $n in the T2T-Q100 regions, $(comm -12 $f.txt q100.txt | wc -l) of them in its truth"
done

# HG003: GIAB v4.2.1 only (no T2T-Q100 truth and no SV truth exist for HG003); q.vcf.gz and aq.vcf.gz as above
pbhappy HG003_GRCh38_1_22_v4.2.1_benchmark.vcf.gz q.vcf.gz --pass-only -r $REF \
    -f HG003_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz --stratification $STRATA --engine=vcfeval --threads 64 -o v421
aardvark compare --preset small -r $REF -t HG003_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    -b HG003_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed.gz -q aq.vcf.gz -o small_v421
```

## Check your results

| Check                                  | 60 Gb                              | 90 Gb                              |
| -------------------------------------- | ---------------------------------- | ---------------------------------- |
| HG002 read set records (command below) | `13cbe0e62e5d3ad87b59121241bf666b` | `0f5f2c935447606145df3158027961fc` |
| HG003 read set records                 | `55805c5ebc7507ccf3e64df9a5cab75e` | `0e61c87330a4534748f159318007f3c8` |
| HG002 aligned BAM records, sorted      | `30f42b8128c5aff6b94cacc572fab4d8` | `ca5d4a33eb1f3d8c5f4dc69b87217162` |
| HG002 pbcall VCF records (A)           | `c76c60bab1ef4d28bab1ceec9b1b8eb0` | `8d5be80e755dcf63d53115d46a068761` |
| HG002 pbcall VCF records (B)           | `b75ddb6295bf901fe9d5f09ac7a08a37` | `ca7192202000ccb29931479a78dd1dff` |
| HG002 pbdv VCF records (RTX PRO 6000)  | `1657c2b71ec0547e68920cafa9ad04eb` | `59fe396836730caab29e68e70027b265` |
| HG002 contigs `hap1.fa` md5            | `5299cd3bef1c3f4e37c1e74d2189ce47` | `48e12f68ea5e32658237bec2fe16fce9` |
| HG002 contigs `hap2.fa` md5            | `31f1908aa8ec9d1a8c4a1f9adf2a91cd` | `8e5e07ee54eff4952fc0ac7cc27bff88` |

```bash
samtools view hg002.60G.reads.bam | md5sum         # BAM records, without the header (also aligned.bam)
zcat hg002.vcf.gz | grep -v '^#' | md5sum          # VCF records (pbcall)
grep -v '^#' small.vcf | md5sum                    # VCF records (pbdv)
```

File md5s depend on the header (command lines, paths), so compare records. pbdv on another GPU model can differ in the
last bits of floating point ([pbdv](pbdv.md)).

[Back to the overview](../README.md)
