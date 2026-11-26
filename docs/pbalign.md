# pbalign

HiFi mapper and aligner. Writes BAM or CRAM, unsorted or sorted with index.

# HiFi Read Alignment

## Human genome

HG002 ([public PacBio Revio data, SPRQ-Nx chemistry](reproduce.md#1-data)): [60 and 90 Gb](reproduce.md#4-read-sets) of HiFi reads aligned to GRCh38.

### Fastest

One AMD EPYC 9755 CPU, 128 cores + SMT, `-j 256`; one warmup run, then 3 timed runs per row, the median ([Method](method.md)).

**Index of GRCh38** (once per reference)

| Preset | Time      | Peak memory | Index  |
| ------ | --------- | ----------- | ------ |
| HIFI   | **2.9 s** | 11 GiB      | 5.3 GB |

**60 Gb**

| Format | Sorted      | Time     | Peak memory | Output  |
| ------ | ----------- | -------- | ----------- | ------- |
| BAM    | no          | **21 s** | 10 GiB      | 29.4 GB |
| BAM    | yes, + BAI  | **27 s** | 60 GiB      | 28.8 GB |
| CRAM   | no          | **21 s** | 21 GiB      | 8.5 GB  |
| CRAM   | yes, + CRAI | **29 s** | 60 GiB      | 8.6 GB  |

**90 Gb**

| Format | Sorted      | Time     | Peak memory | Output  |
| ------ | ----------- | -------- | ----------- | ------- |
| BAM    | no          | **32 s** | 10 GiB      | 43.8 GB |
| BAM    | yes, + BAI  | **40 s** | 75 GiB      | 43.2 GB |
| CRAM   | no          | **32 s** | 22 GiB      | 12.6 GB |
| CRAM   | yes, + CRAI | **43 s** | 75 GiB      | 12.8 GB |

### Cheapest

The sorted BAM + BAI on AWS EC2 m7a.16xlarge spot instances (64 cores, 256 GiB, `-j 64`), us-east-1d, 2026-11-25,
at the spot price of that day ($0.9318/h); the mean of 3 instances each. Compute cost: the pbalign run itself at
the spot price of the instance. The records are the same as those of Fastest above.

| Reads | pbalign time | Compute cost | Peak memory |
| ----- | ------------ | ------------ | ----------- |
| 60 Gb | **59 s**     | **$0.015**   | 57 GiB      |
| 90 Gb | **1:28 min** | **$0.023**   | 73 GiB      |

The whole instance, from launch to shutdown (start, fetch of the reads, index and pbalign from S3 with the AWS CLI's
CRT client, pbalign, upload of the BAM + BAI to S3): 60 Gb 1:45 min ($0.027), 90 Gb 2:23 min ($0.037).

- `--max-memory` sets the memory budget of pbalign (default `auto`: the RAM, the cgroup or the address-space limit).

```bash
pbalign index -j 256 human_GRCh38_no_alt_analysis_set.fasta GRCh38.pai  # once
pbalign align -j 256 --sort GRCh38.pai reads.bam aligned.bam            # without --sort: unsorted
```

Commands for this data set: [Reproduce](reproduce.md).

## Other genomes

Seven more species, from a 0.5 Gb insect genome to the 94 Gb mistletoe genome, with two hexaploid plants, and a human tumor/normal pair. Unsorted BAM, default options, pbalign ed11c12 (the version before beab3d9). The colo829 index is the GRCh38 index above. One AMD EPYC 9755 CPU, 128 cores + SMT, `-j 256`; one warmup run per data set, then one run ([Method](method.md)).

| Set       | Reads  | Genome  | Index | Index memory | Align        | CPU time | Align memory |
| --------- | ------ | ------- | ----- | ------------ | ------------ | -------- | ------------ |
| ladybug   | 42 Gb  | 0.5 Gb  | 0.7 s | 2 GiB        | **46 s**     | 2.4 h    | 12 GiB       |
| oak       | 86 Gb  | 0.8 Gb  | 0.9 s | 4 GiB        | **42 s**     | 2.0 h    | 10 GiB       |
| maize     | 70 Gb  | 2.2 Gb  | 2.6 s | 8 GiB        | **28 s**     | 1.2 h    | 12 GiB       |
| mouse     | 140 Gb | 2.7 Gb  | 2.0 s | 11 GiB       | **1:01 min** | 2.6 h    | 11 GiB       |
| wheat     | 62 Gb  | 14.7 Gb | 11 s  | 50 GiB       | **3:15 min** | 0.7 h    | 32 GiB       |
| redwood   | 31 Gb  | 47.7 Gb | 26 s  | 216 GiB      | **1:34 min** | 0.5 h    | 62 GiB       |
| mistletoe | 104 Gb | 94.3 Gb | 54 s  | 277 GiB      | **47 s**     | 1.4 h    | 160 GiB      |
| colo829   | 260 Gb | 3.1 Gb  | 2.9 s | 11 GiB       | **1:32 min** | 2.7 h    | 10 GiB       |

Data, all public:

| Set       | Genome                                                   | Reference                                                                                   | Size                     | Reads                                                                                                                                                                                     |
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
pbalign index -j 256 ref.fasta ref.pai
pbalign align -j 256 ref.pai reads.fofn aligned.bam  # reads.fofn: the BAM or FASTQ files, one per line
```

# Contig Alignment

## Human genome

The contigs of [pbhifiasm](pbhifiasm.md) of the same HG002 read sets (each haplotype one FASTA) aligned to GRCh38 with
`--preset CONTIG`, pbalign beab3d9.

| Read set | Haplotype | Contigs | Length  | N50     |
| -------- | --------- | ------- | ------- | ------- |
| 60 Gb    | hap1      | 1,629   | 3.01 Gb | 7.6 Mb  |
| 60 Gb    | hap2      | 1,387   | 2.92 Gb | 8.0 Mb  |
| 90 Gb    | hap1      | 313     | 3.07 Gb | 78.6 Mb |
| 90 Gb    | hap2      | 278     | 2.92 Gb | 71.0 Mb |

**Index of GRCh38** (once per reference)

| Preset | Time      | Peak memory | Index  |
| ------ | --------- | ----------- | ------ |
| CONTIG | **3.2 s** | 13 GiB      | 6.7 GB |

### Fastest

Same setup as Fastest above.

**60 Gb**

| Haplotype | Format | Sorted      | Time         | Peak memory | Output  |
| --------- | ------ | ----------- | ------------ | ----------- | ------- |
| hap1      | BAM    | no          | **44 s**     | 77 GiB      | 0.99 GB |
| hap1      | BAM    | yes, + BAI  | **44 s**     | 87 GiB      | 0.99 GB |
| hap1      | CRAM   | no          | **45 s**     | 81 GiB      | 0.20 GB |
| hap1      | CRAM   | yes, + CRAI | **45 s**     | 87 GiB      | 0.20 GB |
| hap2      | BAM    | no          | **1:04 min** | 82 GiB      | 0.94 GB |
| hap2      | BAM    | yes, + BAI  | **1:04 min** | 87 GiB      | 0.94 GB |
| hap2      | CRAM   | no          | **1:04 min** | 89 GiB      | 0.18 GB |
| hap2      | CRAM   | yes, + CRAI | **1:05 min** | 89 GiB      | 0.18 GB |

**90 Gb**

| Haplotype | Format | Sorted      | Time         | Peak memory | Output  |
| --------- | ------ | ----------- | ------------ | ----------- | ------- |
| hap1      | BAM    | no          | **2:30 min** | 130 GiB     | 1.18 GB |
| hap1      | BAM    | yes, + BAI  | **2:29 min** | 126 GiB     | 1.18 GB |
| hap1      | CRAM   | no          | **2:34 min** | 126 GiB     | 0.38 GB |
| hap1      | CRAM   | yes, + CRAI | **2:35 min** | 128 GiB     | 0.38 GB |
| hap2      | BAM    | no          | **1:23 min** | 117 GiB     | 1.11 GB |
| hap2      | BAM    | yes, + BAI  | **1:22 min** | 121 GiB     | 1.11 GB |
| hap2      | CRAM   | no          | **1:27 min** | 119 GiB     | 0.34 GB |
| hap2      | CRAM   | yes, + CRAI | **1:25 min** | 119 GiB     | 0.34 GB |

### Cheapest

AWS EC2 m7a.16xlarge spot instance (64 cores, 256 GiB, `-j 64`, `--max-memory 64G`), sorted BAM, us-east-1d (60 Gb) and
us-east-1c (90 Gb), 2026-11-25, at the spot price of that day ($0.9335/h, $0.7546/h): the instance of the assembly
route, which then runs the lift and [pbcall](pbcall.md#with-the-assembly-and-the-lift). Compute cost in bold; peak
memory after the comma.

| Step | 60 Gb                         | 90 Gb                          |
| ---- | ----------------------------- | ------------------------------ |
| hap1 | 1:11 min (**$0.018**), 66 GiB | 3:36 min (**$0.045**), 119 GiB |
| hap2 | 1:39 min (**$0.026**), 69 GiB | 1:58 min (**$0.025**), 113 GiB |

```bash
pbalign index -j 256 --preset CONTIG human_GRCh38_no_alt_analysis_set.fasta GRCh38.contig.pai  # once
pbalign align -j 256 --preset CONTIG --sort GRCh38.contig.pai hap1.fa hap1.bam                  # without --sort: unsorted
```

# Lift

The reads aligned to GRCh38 and to the sample's own contigs (hap1 + hap2) in one pass: a read that the guards allow
takes its place on the reference through its contig, the others keep their GRCh38 record. The input of the contigs
is the sorted contig BAMs of [Contig Alignment](#contig-alignment); every read is in the output. pbalign beab3d9.

## Human genome

| Reads | Lifted                 | Not on the contigs | Outside the callable regions | Implausible | Without a reference record | Moved between chrX and chrY |
| ----- | ---------------------- | ------------------ | ---------------------------- | ----------- | -------------------------- | --------------------------- |
| 60 Gb | 3,272,551 of 3,695,102 | 124,199            | 289,825                      | 8,524       | 3                          | 0                           |
| 90 Gb | 4,962,510 of 5,542,653 | 242,536            | 324,726                      | 12,878      | 3                          | 0                           |

### Fastest

Same setup as Fastest above.

| Reads | Step                   | Time         | Peak memory | Output   |
| ----- | ---------------------- | ------------ | ----------- | -------- |
| 60 Gb | Index of hap1 + hap2   | **4.0 s**    | 27 GiB      | 11.35 GB |
| 60 Gb | Lift, sorted BAM + BAI | **1:17 min** | 77 GiB      | 28.87 GB |
| 90 Gb | Index of hap1 + hap2   | **4.2 s**    | 25 GiB      | 11.43 GB |
| 90 Gb | Lift, sorted BAM + BAI | **1:42 min** | 94 GiB      | 43.20 GB |

### Cheapest

The same instances as the contigs above, one step after the other. Compute cost in bold.

| Step                   | 60 Gb                         | 90 Gb                         |
| ---------------------- | ----------------------------- | ----------------------------- |
| Index of hap1 + hap2   | 11 s (**$0.003**), 20 GiB     | 13 s (**$0.003**), 20 GiB     |
| Lift, sorted BAM + BAI | 2:28 min (**$0.038**), 61 GiB | 3:38 min (**$0.046**), 64 GiB |

The whole instance, with pbcall after the lift: [pbcall](pbcall.md#with-the-assembly-and-the-lift).

```bash
cat hap1.fa hap2.fa > contigs.fa
pbalign index -j 256 contigs.fa contigs.pai
pbalign align -j 256 --sort --assembly-index contigs.pai --assembly-bam hap1.bam,hap2.bam GRCh38.pai reads.bam lifted.bam
```

Download: release [20261126](../README.md#downloads).

[Back to the overview](../README.md)
