# Data

The raw UAV dataset is not committed to this repository.

Expected local location:

`data/raw/Dataset_T-ITS.csv`

## Important Structure

The source CSV contains several embedded cyber and physical sections
with different schemas.

The reconstructed sections are:

- cyber_benign
- physical_benign
- cyber_dos
- physical_dos
- cyber_replay
- physical_replay
- cyber_evil_twin
- physical_evil_twin
- cyber_fdi
- physical_fdi

The raw CSV must not be treated as one homogeneous machine-learning
table.

## Git Policy

Do not commit:

- raw datasets
- parsed row-level datasets
- window-level datasets
- split-level feature matrices
- large fitted models
- row-level prediction dumps

Commit:

- lightweight JSON configuration files
- audit summaries
- aggregate result tables
- figures
- research documentation
