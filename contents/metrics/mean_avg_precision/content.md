# Load Data in TF.Data

:::{note} 
Benefits of TF-Data on the loading-preprocessing pipeline:
- **Small Dataset**: If all your data can reside in memory, then TF-Data does not help much. Well, at least, it makes the code cleaner.
- **Large Dataset**: Take advantage of multi-core CPU and speed up the pipeline.
:::

## Problem