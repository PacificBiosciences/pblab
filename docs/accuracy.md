# Accuracy

Whole pipelines on HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38. **New**: pbweave, then pbdv and the SV caller. **Old**: pbmm2 v26.2.99, then DeepVariant 1.10.0 (or Parabricks 4.7.1) and sawfish 2.2.1. Best F1 of each group in bold.

## Small variants

GIAB v4.2.1, PASS calls, scored with pbhappy (vcfeval engine). pbdv on the RTX PRO 6000.

| Reads |  Type |         Pipeline         |  Recall | Precision |      F1     |   FN  |   FP  |
| ----- | ----- | ------------------------ | ------- | --------- | ----------- | ----- | ----- |
| 60 Gb | SNP   | new: pbweave + pbdv      | 99.834% | 99.896%   | 99.865%     | 5,599 | 3,516 |
| 60 Gb | SNP   | old: pbmm2 + DeepVariant | 99.835% | 99.897%   | **99.866%** | 5,568 | 3,472 |
| 60 Gb | SNP   | old: pbmm2 + Parabricks  | 99.828% | 99.890%   | 99.859%     | 5,801 | 3,691 |
| 60 Gb | INDEL | new: pbweave + pbdv      | 98.645% | 98.853%   | **98.749%** | 7,118 | 6,238 |
| 60 Gb | INDEL | old: pbmm2 + DeepVariant | 98.631% | 98.845%   | 98.738%     | 7,192 | 6,279 |
| 60 Gb | INDEL | old: pbmm2 + Parabricks  | 98.320% | 98.568%   | 98.444%     | 8,827 | 7,783 |
| 90 Gb | SNP   | new: pbweave + pbdv      | 99.926% | 99.922%   | **99.924%** | 2,504 | 2,617 |
| 90 Gb | SNP   | old: pbmm2 + DeepVariant | 99.922% | 99.926%   | **99.924%** | 2,611 | 2,491 |
| 90 Gb | SNP   | old: pbmm2 + Parabricks  | 99.916% | 99.927%   | 99.922%     | 2,812 | 2,474 |
| 90 Gb | INDEL | new: pbweave + pbdv      | 99.572% | 99.561%   | **99.567%** | 2,249 | 2,393 |
| 90 Gb | INDEL | old: pbmm2 + DeepVariant | 99.562% | 99.559%   | 99.561%     | 2,299 | 2,403 |
| 90 Gb | INDEL | old: pbmm2 + Parabricks  | 99.372% | 99.391%   | 99.381%     | 3,302 | 3,320 |

## Structural variants

NIST T2T HG002 Q100 v1.1 SV benchmark (draft V0.019), scored with pbtruvari: `bench --passonly --pick ac --dup-to-ins`, then `refine`. Numbers from `refine.variant_summary.json`.

| Reads |         Pipeline         | Recall | Precision |     F1     |  FN |  FP |
| ----- | ------------------------ | ------ | --------- | ---------- | --- | --- |
| 60 Gb | new: pbweave + SV caller | 97.09% | 99.57%    | **98.32%** | 729 | 104 |
| 60 Gb | old: pbmm2 + sawfish     | 96.12% | 99.27%    | 97.67%     | 918 | 157 |
| 90 Gb | new: pbweave + SV caller | 97.60% | 99.56%    | **98.57%** | 603 | 108 |
| 90 Gb | old: pbmm2 + sawfish     | 96.57% | 99.24%    | 97.89%     | 810 | 165 |

How to get these numbers yourself: [Reproduce](reproduce.md).

[Back to the overview](../README.md)
