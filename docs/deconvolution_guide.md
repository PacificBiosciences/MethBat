# Deconvolution guide
This section of the user guide contains information on using `methbat deconvolve` to estimate cell type proportions from methylation data using a reference atlas.
This approach uses non-negative least squares (NNLS) regression to deconvolve bulk methylation data into constituent cell types based on a pre-defined atlas of cell type-specific methylation patterns.

Table of contents:

* [Quickstart](#quickstart)
* [Atlas format](#atlas-format)
* [Output files](#output-files)

# Quickstart
The following command will perform cell type deconvolution on a single dataset:

```bash
methbat deconvolve \
    --input-prefix {IN_PREFIX} \
    --atlas-regions {ATLAS_FILE} \
    --output-estimate {OUT_JSON}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus; supports both uncompressed (.bed) and compressed (.bed.gz) files
* `--atlas-regions {ATLAS_FILE}` - the cell type atlas file in MethBat format; the format is [described below](#atlas-format) and example atlases are provided in the [data folder](../data/cell_atlas/)
* `--output-estimate {OUT_JSON}` - the output JSON file containing deconvolution results


## Atlas format
The cell type atlas must be a tab-separated file with the following structure:

### Required columns:
* `chr` - chromosome name; `chrom` or `chromosome` are also accepted as column headers
* `start` - 0-based start coordinate (inclusive)
* `end` - 0-based end coordinate (exclusive)
* Additional columns for each cell type with methylation values (0.0 to 1.0), where 0.0 is fully unmethylated and 1.0 is fully methylated

### Example atlas format:
```
chr	start	end	adipocyte	endothelial_cell	colon_fibroblasts	heart_fibroblasts	...
chr1	1066168	1066385	0.71	0.81	0.78	0.80	...
chr1	1071849	1072048	0.86	0.86	0.79	0.86	...
chr1	1179392	1179548	0.88	0.91	0.76	0.83	...
...
```

### Available atlases
Pre-formatted atlases are available in the [data folder](../data/cell_atlas/).

# Output files
The deconvolution process generates a single JSON output file with the following structure:

```json
{
  "version": "0.16.0-8dcb2bf",
  "atlas_md5sum": "6e30a2eafd8da7410d4e7766c7c71d94",
  "cli_settings": {
    "input_prefix": "./pipeline/cpg_5mc_model/HG001",
    "atlas_regions": "./data/cell_atlas/nanomix/39Bisulfite.methbat.tsv",
    "output_estimate": "./HG001_cell_estimate.json",
    "min_active": 0.05
  },
  "metrics": {
    "normalized_fraction": {
      "B-cell": 0.8549362852865744,
      "NK-cell": 0.0,
      ...
    },
    "nnls_raw_fraction": {
      "B-cell": 0.7578555187460534,
      "NK-cell": 0.0,
      ...
    },
    "nnls_residual_error": 12.898144132607202,
    "residual": [
      -0.02371285749724783,
      ...
    ]
  }
}
```

## Field descriptions:
* `version` - MethBat version used for deconvolution
* `atlas_md5sum` - MD5 checksum of the atlas file for verification of exact atlas file
* `cli_settings` - Copy of the command-line parameters used
* `metrics` - Deconvolution results and statistics:
  * `normalized_fraction` - Normalized cell type proportions (sum to 1.0)
  * `nnls_raw_fraction` - Raw cell type proportions from NNLS regression
  * `nnls_residual_error` - Residual error from the NNLS regression
  * `residual` - Array of residual values for each region used in deconvolution. This can be useful in identifying regions that are prone to high error rates across multiple datasets.

