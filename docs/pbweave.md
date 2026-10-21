# pbweave

HiFi mapper and aligner. Writes BAM or CRAM, unsorted or sorted with index.

Numbers, except [Other genomes](#other-genomes): HG002 (PacBio Revio, SPRQ-Nx chemistry, public), 60 and 90 Gb of HiFi reads, GRCh38.

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

## Other genomes

Seven more species, from a 0.5 Gb insect genome to the 94 Gb mistletoe genome, with two hexaploid plants, and a human tumor/normal pair. Unsorted BAM, default options. One AMD EPYC 9755 NUMA node, 128 cores + SMT, `-j 256`, one run each, alone on its node; reads and index read into memory before the run.

|    Set    | Reads  |  Genome |  Index   | Index memory |    Align     | Faster than pbmm2 | CPU time | Align memory |
| --------- | ------ | ------- | -------- | ------------ | ------------ | ----------------- | -------- | ------------ |
| ladybug   | 42 Gb  | 0.5 Gb  | 0.8 s    | 3 GB         | **43 s**     | 29x               | 2.2 h    | 13 GB        |
| oak       | 86 Gb  | 0.8 Gb  | 1.2 s    | 4 GB         | **39 s**     | 31x               | 1.8 h    | 10 GB        |
| maize     | 70 Gb  | 2.2 Gb  | 3.3 s    | 9 GB         | **27 s**     | 41x               | 1.1 h    | 13 GB        |
| mouse     | 140 Gb | 2.7 Gb  | 2.9 s    | 9 GB         | **53 s**     | 46x               | 2.1 h    | 11 GB        |
| wheat     | 62 Gb  | 14.7 Gb | 15 s     | 50 GB        | **3:18 min** | 5x                | 0.7 h    | 32 GB        |
| redwood   | 31 Gb  | 47.7 Gb | 56 s     | 207 GB       | **1:37 min** | 5x                | 0.4 h    | 61 GB        |
| mistletoe | 104 Gb | 94.3 Gb | 1:58 min | 290 GB       | **47 s**     | 23x               | 1.4 h    | 163 GB       |
| colo829   | 260 Gb | 3.1 Gb  | 3.6 s    | 11 GB        | **1:30 min** | 35x               | 2.6 h    | 10 GB        |

For reference, pbmm2 v26.2.99 (`--preset HIFI`, `-j 256`) on the same node:

|    Set    |     Align     | CPU time | Peak memory |
| --------- | ------------- | -------- | ----------- |
| ladybug   | 20:50 min     | 62 h     | 33 GB       |
| oak       | 20:01 min     | 51 h     | 50 GB       |
| maize     | 18:19 min     | 59 h     | 21 GB       |
| mouse     | 40:22 min (1) | 105 h    | 68 GB       |
| wheat     | 15:12 min     | 23 h     | 41 GB       |
| redwood   | 8:11 min      | 11 h     | 103 GB      |
| mistletoe | 18:02 min     | 24 h     | 193 GB      |
| colo829   | 52:02 min     | 95 h     | 54 GB       |

(1) With mimalloc v3.5.3 (`LD_PRELOAD`). The same binary with its default allocator (glibc malloc) takes 1:56 h on mouse, with the same alignments: most of its 256 threads wait on the kernel's mmap lock. The other sets ran with glibc malloc; their pbmm2 times may be inflated the same way, less so.

How often pbweave agrees with pbmm2. pbmm2 is not the truth here; on these genomes there is none.

|    Set    | Same primary locus (pbmm2 MAPQ >= 5) | Bases pbmm2 aligns, pbweave not | Bases pbweave aligns, pbmm2 not | pbweave MAPQ 60, pbmm2 elsewhere |
| --------- | ------------------------------------ | ------------------------------- | ------------------------------- | -------------------------------- |
| ladybug   | 93.34%                               | 2.817%                          | 8.593%                          | 3.025%                           |
| oak       | 98.14%                               | 0.837%                          | 1.937%                          | 0.926%                           |
| maize     | 99.86%                               | 0.100%                          | 0.026%                          | 0.009%                           |
| mouse     | 99.31%                               | 1.185%                          | 0.072%                          | 0.431%                           |
| wheat     | 99.99%                               | 0.017%                          | 0.005%                          | 0.003%                           |
| redwood   | 99.84%                               | 0.034%                          | 0.038%                          | 0.068%                           |
| mistletoe | 99.13%                               | 0.542%                          | 0.160%                          | 0.117%                           |
| colo829   | 99.46%                               | 0.459%                          | 0.255%                          | 0.234%                           |

- Same primary locus: of the reads whose pbmm2 primary has MAPQ >= 5, the share with a pbweave primary that overlaps it. Bases: read bases in an alignment of one tool and in none of the other, of all read bases.
- Ladybug and oak differ most: pbweave aligns 8.593% and 1.937% of all their read bases that pbmm2 leaves unaligned, and puts 93.34% of the ladybug reads with pbmm2 MAPQ >= 5 at pbmm2's locus.

Data, all public:

|    Set    |                          Genome                          |                                          Reference                                          |           Size           |                                                                                           Reads                                                                                           |
| --------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ladybug   | two-spot ladybird (*Adalia bipunctata*)                  | [icAdaBipu1.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_910592335.1/)               | 0.48 Gb, 119 contigs     | Revio, 2 SMRT Cells ([1](https://downloads.pacbcloud.com/public/revio/2023Q1/ladybug-rep1/), [2](https://downloads.pacbcloud.com/public/revio/2023Q1/ladybug-rep2/))                      |
| oak       | pedunculate oak (*Quercus robur*)                        | [dhQueRobu3.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_932294415.1/)               | 0.79 Gb, 97 contigs      | Revio, 2 SMRT Cells ([1](https://downloads.pacbcloud.com/public/revio/2023Q1/oak-rep1/), [2](https://downloads.pacbcloud.com/public/revio/2023Q1/oak-rep2/))                              |
| maize     | maize B73 (*Zea mays*)                                   | [Zm-B73-REFERENCE-NAM-5.0](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_902167145.1/)   | 2.18 Gb, 687 contigs     | Revio, 1 SMRT Cell ([1](https://downloads.pacbcloud.com/public/revio/2023Q1/maize-B73-rep1/))                                                                                             |
| mouse     | mouse (*Mus musculus*)                                   | [mm39](https://hgdownload.soe.ucsc.edu/goldenPath/mm39/bigZips/)                            | 2.73 Gb, 61 contigs      | Revio, 2 SMRT Cells ([1](https://downloads.pacbcloud.com/public/revio/2023Q1/mouse-rep1/), [2](https://downloads.pacbcloud.com/public/revio/2023Q1/mouse-rep2/))                          |
| wheat     | bread wheat cv. Fielder (*Triticum aestivum*, hexaploid) | [Fielder v1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_907166925.1/)                 | 14.70 Gb, 3,795 contigs  | Sequel II HiFi, ENA ERR5863232 + ERR5863236 ([1](https://www.ebi.ac.uk/ena/browser/view/ERR5863232), [2](https://www.ebi.ac.uk/ena/browser/view/ERR5863236))                              |
| redwood   | coast redwood (*Sequoia sempervirens*, hexaploid)        | [redwood 2020 primary contigs](https://downloads.pacbcloud.com/public/dataset/redwood2020/) | 47.75 Gb, 46,991 contigs | Sequel II HiFi, SRA SRR11211695 ([1](https://www.ncbi.nlm.nih.gov/sra/SRR11211695))                                                                                                       |
| mistletoe | European mistletoe (*Viscum album*)                      | [drVisAlbu1.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_963277665.1/)               | 94.26 Gb, 5,096 contigs  | Revio, 2 SMRT Cells, 2 barcodes each ([1](https://downloads.pacbcloud.com/public/revio/2023Q1/mistletoe-rep1/), [2](https://downloads.pacbcloud.com/public/revio/2023Q1/mistletoe-rep2/)) |
| colo829   | human: COLO829 tumor + COLO829BL normal, in one run      | GRCh38 no-alt analysis set                                                                  | 3.10 Gb, 195 contigs     | Revio, 1 SMRT Cell each ([1](https://downloads.pacbcloud.com/public/revio/2024Q4/WGS/COLO829/COLO829/), [2](https://downloads.pacbcloud.com/public/revio/2024Q4/WGS/COLO829/COLO829BL/))  |

```bash
pbweave index -j 256 ref.fasta ref.wvi
pbweave align -j 256 ref.wvi reads.fofn aligned.bam      # reads.fofn: the BAM or FASTQ files, one per line
```

[Back to the overview](../README.md)
