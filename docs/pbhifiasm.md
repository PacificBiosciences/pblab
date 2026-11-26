# pbhifiasm

HiFi assembler: a reimplementation of hifiasm, tuned for speed. Reads HiFi BAM, FASTA or FASTQ; writes the contigs of
each haplotype as GFA (as hifiasm: `<prefix>.bp.hap1.p_ctg.gfa`, `<prefix>.bp.hap2.p_ctg.gfa`).

## Human genome

HG002 ([public PacBio Revio data, SPRQ-Nx chemistry](reproduce.md#1-data)): [60 and 90 Gb](reproduce.md#4-read-sets) of HiFi reads.

### Fastest

One AMD EPYC 9755 CPU, 128 cores + SMT, `-j 256`; one warmup run per data set, then one run
([Method](method.md)).

| Reads | Time         | CPU time | Peak memory |
| ----- | ------------ | -------- | ----------- |
| 60 Gb | **2:12 min** | 8.0 h    | 85 GiB      |
| 90 Gb | **4:18 min** | 16.4 h   | 125 GiB     |

The contigs (the warmup run gave the same):

| Reads | Haplotype | Contigs | Length  | N50     |
| ----- | --------- | ------- | ------- | ------- |
| 60 Gb | hap1      | 1,629   | 3.01 Gb | 7.6 Mb  |
| 60 Gb | hap2      | 1,387   | 2.92 Gb | 8.0 Mb  |
| 90 Gb | hap1      | 313     | 3.07 Gb | 78.6 Mb |
| 90 Gb | hap2      | 278     | 2.92 Gb | 71.0 Mb |

### Cheapest

AWS EC2 m7a.16xlarge spot instance (64 cores, 256 GiB, `-j 64`), us-east-1b, 2026-11-25, at the spot price of
that day ($0.987/h). Compute cost: the pbhifiasm run itself at the spot price of the instance. The contigs are the
same as those of Fastest above (the same md5s).

| Reads | pbhifiasm time | Compute cost | Peak memory |
| ----- | -------------- | ------------ | ----------- |
| 60 Gb | **6:14 min**   | **$0.102**   | 78 GiB      |
| 90 Gb | **12:24 min**  | **$0.204**   | 115 GiB     |

The whole instance, from launch to shutdown (start, fetch of the reads and pbhifiasm from S3, assembly, upload of the
contig GFAs to S3): 60 Gb 7:59 min ($0.131), 90 Gb 14:16 min ($0.235).

The contigs on GRCh38: [pbalign, Contig Alignment](pbalign.md#contig-alignment).

## Reproduce

The read sets: [Reproduce](reproduce.md), sections 1 and 4.

```bash
pbhifiasm asm -j 256 -o hg002.60G hg002.60G.reads.bam     # writes hg002.60G.bp.hap{1,2}.p_ctg.gfa
for h in hap1 hap2; do                                    # the contigs of each haplotype as FASTA
    awk -F'\t' '$1 == "S" {print ">"$2; print $3}' hg002.60G.bp.$h.p_ctg.gfa > hg002.60G.$h.fa
done
md5sum hg002.60G.hap1.fa hg002.60G.hap2.fa
```

| Check         | 60 Gb                              | 90 Gb                              |
| ------------- | ---------------------------------- | ---------------------------------- |
| `hap1.fa` md5 | `5299cd3bef1c3f4e37c1e74d2189ce47` | `48e12f68ea5e32658237bec2fe16fce9` |
| `hap2.fa` md5 | `31f1908aa8ec9d1a8c4a1f9adf2a91cd` | `8e5e07ee54eff4952fc0ac7cc27bff88` |

The same reads and the same binary give the same contigs: the warmup and the timed run of Fastest, and the run of Cheapest with 64 threads, match.

[Back to the overview](../README.md)
