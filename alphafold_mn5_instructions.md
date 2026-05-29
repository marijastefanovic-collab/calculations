# Running AlphaFold on MareNostrum 5

## Requirements

- `prepare_proteins` and `bsc_calculations` installed in your Python environment
- SSH access to MN5 (`bsc395943@alogin4.bsc.es`)
- Working directory on MN5: `/gpfs/projects/bsc72/marija_stefanovic/`

## Step 1 — Prepare files locally (Jupyter Notebook)

```python
import prepare_proteins
import bsc_calculations

# Load sequences from FASTA file
sequences = prepare_proteins.sequenceModels('your_sequences.fasta')

# Generate AlphaFold input files and commands
jobs = sequences.setUpAlphaFold('alphafold')

# Generate SLURM script for MN5 accelerated partition
bsc_calculations.mn5.jobArrays(jobs, job_name='AF_sequences', partition='acc_bscls', program='alphafold')
```

This creates:
- `alphafold/` — folder with input sequences and output directory
- `slurm_array.sh` — SLURM submission script

## Step 2 — Upload to MN5

```bash
rsync -avz alphafold bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/
rsync -avz slurm_array.sh bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/
```

## Step 3 — Submit the job

```bash
ssh bsc395943@alogin4.bsc.es
cd /gpfs/projects/bsc72/marija_stefanovic/
sbatch slurm_array.sh
```

## Step 4 — Monitor

```bash
squeue -u bsc395943
```

## Step 5 — Download results when finished

Pack only the relaxed PDB files on the cluster:
```bash
tar cvf AF_sequences.tar alphafold/output_models/*/relaxed_model_*pdb
```

Then download to your local machine:
```bash
rsync -avz bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/AF_sequences.tar /home/ordi/marija_calculations/
```

## Notes

- AlphaFold version used: `2.3.2`
- Partition: `acc_bscls` (accelerated, up to 48h)
- Account: `bsc72`
- Do NOT run jobs from `/gpfs/home` — use `/gpfs/projects/bsc72/` instead
- DSSP is installed as `mkdssp` on modern systems
