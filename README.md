# Snakemake with SLURM on MPCDF Raven Cluster

This repository demonstrates a Snakemake workflow with three stages:
1. **Generate** - Create random numbers in parallel jobs
2. **Gather** - Combine all generated files
3. **Sort** - Sort the combined numbers

Clone this repository. Set up `pixi`:

Run tasks:

```shell
# Local execution
pixi run local

# SLURM execution
pixi run slurm

# SLURM with Apptainer containers
pixi run slurm_apptainer

# Local with Apptainer containers
pixi run apptainer
```

## How to start from scratch

Set up `pixi` as shown above. Then initialise the workspace:

```shell
pixi init snakemake-psi-tier3-example
cd snakemake-psi-tier3-example
pixi workspace channel add bioconda
pixi add snakemake
pixi add snakemake-executor-plugin-slurm
```

Generate partition file (**this has to be done for each cluster**):

```
generate-slurm-partition-config -o partitions.yaml
```

Please also mind that the `profiles/config.yaml` needs to be adjusted to the queue
names and slurm user of the given cluster:

```
# Workflow profile for SLURM executor
# This file configures how Snakemake interacts with the SLURM cluster

executor: slurm
latency-wait: 30  # Wait time (seconds) for output files to appear on shared filesystem
slurm-logdir: "${HOME}/.snakemake/slurm_logs"

# Default resources for all SLURM jobs
default-resources:
  slurm_partition: small  # Default SLURM partition
  slurm_account: mpp_cpu       # SLURM account for job submission
  runtime: 10
```

To test the local workflow:

```shell
pixi shell
snakemake --verbose --cores 1 --printshellcmds --snakefile workflow/Snakefile sort_script
```

To test the SLURM workflow (assuming you're still in the `pixi shell`):

```shell
rm results/sorted_numbers_script.txt
export SNAKEMAKE_SLURM_PARTITIONS=${PWD}/partitions.yaml
snakemake --executor slurm \
  --workflow-profile ${PWD}/profiles \
  -j unlimited \
  --verbose \
  --printshellcmds \
  --snakefile workflow/Snakefile sort_script
```

## Configuration

The workflow parameters can be adjusted in `config.yaml`:
- `num_files`: Number of parallel jobs to generate random numbers (default: 5)
- `numbers_per_file`: Random numbers generated per file (default: 10)

SLURM executor settings are in `profiles/config.yaml`.

## Workflow Structure

```
generate_numbers (parallel)
    ↓
    data/generated/numbers_0.txt
    data/generated/numbers_1.txt
    ...
    data/generated/numbers_N.txt
    ↓
gather_numbers
    ↓
    data/numbers_gathered.txt
    ↓
sort_script (or sort_local)
    ↓
    results/sorted_numbers_script.txt
```

## Cleaning Up

To remove generated files and start fresh:
```shell
rm -rf results/ data/generated/ data/numbers_gathered.txt .snakemake/
```
