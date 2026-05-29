# Running AlphaFold 3 on MareNostrum 5 — Protein + Peptide + Calcium Ion

## Requirements

- `prepare_proteins` and `bsc_calculations` installed in your Python environment
- SSH access to MN5 (`bsc395943@alogin4.bsc.es`)
- Working directory on MN5: `/gpfs/projects/bsc72/marija_stefanovic/`

## Step 1 — Prepare files locally (Jupyter Notebook)

```python
import prepare_proteins
import bsc_calculations
from Bio import SeqIO

# Read protein sequence from FASTA file
record = list(SeqIO.parse('your_sequence.fasta', 'fasta'))[0]
protein_seq = str(record.seq)

# Initialize with protein as chain A and peptide as chain B
# Note: peptide is a second protein chain, NOT a ligand
sequences = prepare_proteins.sequenceModels({
    'model_name': {
        'A': protein_seq,
        'B': 'PQPQ'        # replace with your peptide sequence
    }
})

# Set up AF3 jobs — calcium ion CA goes in ligands
jobs = sequences.setUpAlphaFold3(
    job_folder='af3_output',
    ligands=['CA'],         # calcium ion CCD code
    model_seeds=[1, 2, 3, 4, 5],
    skip_finished=True,
)

print(f'Prepared {len(jobs)} AF3 jobs')

# Generate SLURM script for MN5 accelerated partition
bsc_calculations.mn5.jobArrays(
    jobs,
    script_name='slurm_array_af3.sh',
    job_name='AF3_job',
    output='AF3_job',
    partition='acc_bscls',
    program='alphafold3',
    gpus=1,
)
```

This creates:
- `af3_output/` — folder with AF3 JSON inputs per model
- `slurm_array_af3.sh` — SLURM submission script

## Step 2 — Upload to MN5

```bash
rsync -avz af3_output bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/
rsync -avz slurm_array_af3.sh bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/
```

## Step 3 — Submit the job

```bash
ssh bsc395943@alogin4.bsc.es
cd /gpfs/projects/bsc72/marija_stefanovic/
sbatch slurm_array_af3.sh
```

## Step 4 — Monitor

```bash
squeue -u bsc395943
```

## Step 5 — Download results when finished

Pack only the top model CIF files on the cluster:
```bash
tar cvf af3_results.tar af3_output/*/output/
```

Then download to your local machine:
```bash
rsync -avz bsc395943@alogin4.bsc.es:/gpfs/projects/bsc72/marija_stefanovic/af3_results.tar /home/ordi/marija_calculations/
```

## Step 6 — Collect results locally (Jupyter Notebook)

```python
scores_df, selected_df, copied_paths = sequences.collectAlphaFold3Results(
    af_folder='af3_output',
    output_folder='af3_top_models',
    metric='ranking_score',
    top_models=5,
    append_model_index=True,
    return_selected=True,
)

print(scores_df.head())
```

## Key notes

- **Peptide** is a protein chain (`'B': 'PQPQ'`), not a ligand
- **Calcium ion** CCD code is `CA` — goes in `ligands=['CA']`
- **AlphaFold 3** is different from AF2 — uses `setUpAlphaFold3` and `program='alphafold3'`
- **Partition:** `acc_bscls` (MN5 accelerated, up to 48h)
- **Account:** `bsc72`
- Do NOT run jobs from `/gpfs/home` — use `/gpfs/projects/bsc72/` instead
