# Accuracy

HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38; a second genome:
[HG003](#hg003). Three workflows:

- **A**: pbalign → pbcall
- **B**: pbhifiasm → contigs → lift → pbcall
- **C**: pbalign → pbdv

> **pbcall is experimental**: it is not part of release [20261126](../README.md#downloads). Workflows A and B show what it
> does today; workflow C uses released tools only.

pbcall ran with `--haploid-xy` and its panel; B with the lifted reads and `--asm-bam` ([pbcall](pbcall.md)). C is the
pbdv VCF of the Fastest run (RTX PRO 6000). Best F1 of each
row group in bold. The VCFs, truths and commands: [Reproduce](reproduce.md).

## Small variants

PASS calls, scored with pbhappy (vcfeval engine, the 127 GIAB v2.0 stratifications). pbcall writes the chrX and chrY
calls outside the PARs haploid; the scored VCFs have them written as homozygous (diploid), as hap.py expects for a male
sample ([Reproduce](reproduce.md#6-scoring)).

### T2T-Q100 v1.1

The small-variant benchmark (smvar), the more complete truth; pbhappy with `--star-as-ref --gender male`.

**60 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN     | FP     |
| ----- | -------- | ------- | --------- | ----------- | ------ | ------ |
| SNP   | A        | 99.605% | 99.835%   | 99.720%     | 14,428 | 6,046  |
| SNP   | B        | 99.780% | 99.889%   | **99.835%** | 8,030  | 4,054  |
| SNP   | C        | 99.536% | 99.694%   | 99.615%     | 16,936 | 11,186 |
| INDEL | A        | 97.586% | 98.029%   | 97.807%     | 20,893 | 17,949 |
| INDEL | B        | 97.851% | 98.078%   | **97.964%** | 18,597 | 17,569 |
| INDEL | C        | 94.908% | 96.197%   | 95.548%     | 44,067 | 34,373 |

**90 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN     | FP     |
| ----- | -------- | ------- | --------- | ----------- | ------ | ------ |
| SNP   | A        | 99.720% | 99.874%   | 99.797%     | 10,236 | 4,620  |
| SNP   | B        | 99.859% | 99.941%   | **99.900%** | 5,141  | 2,174  |
| SNP   | C        | 99.666% | 99.768%   | 99.717%     | 12,203 | 8,489  |
| INDEL | A        | 98.672% | 98.816%   | 98.744%     | 11,490 | 10,833 |
| INDEL | B        | 98.847% | 98.881%   | **98.864%** | 9,975  | 10,262 |
| INDEL | C        | 97.249% | 97.913%   | 97.580%     | 23,808 | 19,046 |

### GIAB v4.2.1

The noinconsistent BED, chr1–22.

Against GIAB v4.2.1, A is slightly ahead of B. The two truths disagree on the calls where A and B differ: of the PASS
SNPs that only B calls (inside both benchmark regions), T2T-Q100 confirms 97% at 60 Gb and 98% at 90 Gb; of those that
only A calls, 22% and 11% ([Reproduce](reproduce.md#6-scoring)). GIAB v4.2.1 was built mostly from reads aligned to
GRCh38 ([Wagner et al. 2022](https://doi.org/10.1016/j.xgen.2022.100128)); B places the reads through the sample's own
assembly.

**60 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN    | FP    |
| ----- | -------- | ------- | --------- | ----------- | ----- | ----- |
| SNP   | A        | 99.845% | 99.956%   | **99.900%** | 5,223 | 1,494 |
| SNP   | B        | 99.832% | 99.893%   | 99.863%     | 5,662 | 3,587 |
| SNP   | C        | 99.840% | 99.894%   | 99.867%     | 5,391 | 3,572 |
| INDEL | A        | 99.480% | 99.547%   | **99.513%** | 2,735 | 2,467 |
| INDEL | B        | 99.494% | 99.508%   | 99.501%     | 2,660 | 2,681 |
| INDEL | C        | 98.645% | 98.819%   | 98.732%     | 7,119 | 6,428 |

**90 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN    | FP    |
| ----- | -------- | ------- | --------- | ----------- | ----- | ----- |
| SNP   | A        | 99.928% | 99.961%   | **99.944%** | 2,435 | 1,316 |
| SNP   | B        | 99.901% | 99.907%   | 99.904%     | 3,346 | 3,146 |
| SNP   | C        | 99.926% | 99.925%   | 99.925%     | 2,496 | 2,541 |
| INDEL | A        | 99.821% | 99.795%   | **99.808%** | 939   | 1,119 |
| INDEL | B        | 99.826% | 99.766%   | 99.796%     | 912   | 1,274 |
| INDEL | C        | 99.587% | 99.566%   | 99.576%     | 2,168 | 2,371 |

## Structural variants

T2T-Q100 v1.1 SV benchmark (stvar, draft V0.019), scored with pbtruvari: `bench --passonly --pick ac --dup-to-ins`, then
`refine` (multi-allelic records split first). Numbers from `refine.variant_summary.json`. refine counts the truth in
the regions it rescored, so the truth total differs a little by workflow (60 Gb: A 24,782, B 26,793). pbdv calls no
SVs.

**60 Gb**

| Workflow | Recall | Precision | F1         | FN  | FP |
| -------- | ------ | --------- | ---------- | --- | -- |
| A        | 98.60% | 99.92%    | 99.25%     | 347 | 20 |
| B        | 99.31% | 99.83%    | **99.57%** | 186 | 46 |

**90 Gb**

| Workflow | Recall | Precision | F1         | FN  | FP |
| -------- | ------ | --------- | ---------- | --- | -- |
| A        | 99.04% | 99.94%    | 99.48%     | 239 | 16 |
| B        | 99.50% | 99.90%    | **99.70%** | 135 | 26 |

## aardvark

[aardvark](https://github.com/PacificBiosciences/aardvark) v1.2.0, all PASS calls without symbolic alleles, split into
biallelic records: `compare --preset small` against GIAB v4.2.1 and T2T-Q100 v1.1 smvar, and `--preset large` against the
complete T2T-Q100 v1.1 ("full": small variants and SVs). F1 of all variants; the best F1 of each column in bold. GT
scores each variant with its genotype, with FN + FP variants in brackets; BASEPAIR scores the bases of the haplotypes,
with FN + FP bases in brackets. The "Small" columns are GT. pbdv calls no SVs, so its full scores miss them.

**60 Gb**

| Workflow | Small, GIAB v4.2.1   | Small, T2T-Q100      | Full, GT             | Full, BASEPAIR          |
| -------- | -------------------- | -------------------- | -------------------- | ----------------------- |
| A        | **99.862%** (10,849) | 99.420% (53,437)     | 99.146% (81,200)     | 90.590% (10,634,994)    |
| B        | 99.830% (13,355)     | **99.544%** (41,950) | **99.382%** (58,673) | **94.157%** (6,908,962) |
| C        | 99.753% (19,421)     | 98.993% (92,621)     | 97.957% (193,553)    | 55.590% (37,625,160)    |

**90 Gb**

| Workflow | Small, GIAB v4.2.1  | Small, T2T-Q100      | Full, GT             | Full, BASEPAIR          |
| -------- | ------------------- | -------------------- | -------------------- | ----------------------- |
| A        | **99.932%** (5,336) | 99.639% (33,239)     | 99.429% (54,397)     | 91.334% (9,827,410)     |
| B        | 99.898% (8,038)     | **99.745%** (23,505) | **99.626%** (35,509) | **95.023%** (5,962,156) |
| C        | 99.892% (8,458)     | 99.390% (56,191)     | 98.349% (156,725)    | 56.272% (37,220,282)    |

## HG003

HG003, the father of HG002 (PacBio Revio, SPRQ chemistry, public), is the held-out genome, as usual in variant-calling
benchmarks. 60 and 90 Gb of HiFi reads, the same commands as for HG002 ([Reproduce](reproduce.md)); pbalign
4e722c5, pbdv on the RTX PRO 6000 ([Method](method.md)). The truth is GIAB v4.2.1 (the noinconsistent BED, chr1–22): T2T-Q100 has no truth for HG003, and GIAB has no SV truth for
it.

### Small variants, GIAB v4.2.1

pbhappy as above. As for HG002, A is ahead of B for SNPs against GIAB v4.2.1; for indels they are even (B ahead by
0.001 at 60 Gb, the same F1 at 90 Gb). Without a T2T-Q100 truth, HG003 cannot check the calls where they differ.

**60 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN    | FP    |
| ----- | -------- | ------- | --------- | ----------- | ----- | ----- |
| SNP   | A        | 99.708% | 99.887%   | **99.797%** | 9,726 | 3,754 |
| SNP   | B        | 99.710% | 99.837%   | 99.774%     | 9,635 | 5,435 |
| SNP   | C        | 99.715% | 99.827%   | 99.771%     | 9,467 | 5,750 |
| INDEL | A        | 99.243% | 99.393%   | 99.318%     | 3,818 | 3,168 |
| INDEL | B        | 99.264% | 99.375%   | **99.319%** | 3,714 | 3,266 |
| INDEL | C        | 98.219% | 98.446%   | 98.332%     | 8,985 | 8,110 |

**90 Gb**

| Type  | Workflow | Recall  | Precision | F1          | FN    | FP    |
| ----- | -------- | ------- | --------- | ----------- | ----- | ----- |
| SNP   | A        | 99.812% | 99.894%   | **99.853%** | 6,252 | 3,536 |
| SNP   | B        | 99.814% | 99.843%   | 99.828%     | 6,182 | 5,244 |
| SNP   | C        | 99.820% | 99.858%   | 99.839%     | 5,981 | 4,727 |
| INDEL | A        | 99.581% | 99.617%   | **99.599%** | 2,114 | 2,002 |
| INDEL | B        | 99.598% | 99.600%   | **99.599%** | 2,029 | 2,095 |
| INDEL | C        | 99.247% | 99.246%   | 99.246%     | 3,800 | 3,949 |

### aardvark

`compare --preset small` against GIAB v4.2.1, GT, as above.

| Workflow | 60 Gb                | 90 Gb                |
| -------- | -------------------- | -------------------- |
| A        | **99.755%** (18,942) | **99.831%** (13,079) |
| B        | 99.735% (20,488)     | 99.810% (14,701)     |
| C        | 99.628% (28,721)     | 99.781% (16,919)     |

[Back to the overview](../README.md)
