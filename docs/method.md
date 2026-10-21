# Method

## Hardware

- AMD EPYC 9755 server: 2 NUMA nodes of 128 cores each, SMT, local NVMe, NVIDIA RTX PRO 6000 Blackwell Server
  Edition on NUMA node 1.
- Intel Xeon Gold 5420+ server with NVIDIA L4 GPUs.

## Setups

- **Fastest** (the Tools table): one whole NUMA node, 128 cores + SMT (256 vCPU), 256 threads, no memory
  limit; each tool alone on that node, the other node idle. pbdv: all cores of the GPU's node with `-j 128`,
  measured while the other node ran pbweave and pbmm2.
- **omics.c.32xlarge** (128 vCPU): 64 cores with their SMT siblings on one NUMA node, memory on that node;
  pbmm2, DeepVariant, sawfish and the SV caller under a 256 GiB hard memory limit.
- **omics.g6.4xlarge** (16 vCPU, 1 L4): 8 cores + SMT on the L4 server, the L4 on the same NUMA node,
  60 GiB hard memory limit.
- Cloud CPUs can be slower than these servers; the times are of emulated instances.

## Runs

- One run per NUMA node at a time. Cloud setups: both nodes busy (pbweave to an unsorted BAM: the other node
  idle).
- New tools: 1 warmup, then 1 timed run; the timed run's output was scored. Old tools: 1 run.
- Time is the wall clock of the whole command; peak memory is max RSS (SV caller on omics.c.32xlarge: not
  shown, its memory limit counts page cache).
- Cost = time x on-demand price per hour of the instance (AWS HealthOmics, us-east-1, price list 2026-09-11,
  compute only).
- Measured 2026-10-21. Versions: pbweave c748e35, pbdv 9a28b44, pbhappy 86114da, pbtruvari f64e269,
  pbsamoa 191c09e, SV caller 759e689 (Research Experiment; all 26.3.99).
- The release binaries are the measured ones, except pbdv and pbsamoa: these were rebuilt afterwards with a
  clean version string (pbdv: its parent commit fe9793a differs only in a build symlink). Rebuilt, they give
  byte-identical output: the same pbdv VCF and gVCF and the same read sets on both data sets.

[Back to the overview](../README.md)
