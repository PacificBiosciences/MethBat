# Data
This folder contains data resources for use with MethBat. Subfolders:

* [cell_atlas](./cell_atlas/) - Cell atlases that have been pre-configured to work with `methbat deconvolve`
* [report_regions](./report_regions/) - Regions with known methylation patterns that have been pre-configured to work with `methbat report`

## Region / cohort profiles
These files contain background / cohort methylation profiles that can be provided to MethBat to describe coordinates of regions of interest (for example CpG islands).
Optional per-region labels may appear under the column `region_label` in newer outputs; legacy tables may still use `cpg_label`, which MethBat accepts when reading.
Note that methylation profiles can be tissue specific depending on the region. 
For example, running a blood sample dataset against the our HPRC-based (cell-line based) background profile is likely to lead to more false positive / negative results with respect to comparison classifications.
These cohort profiles are subject to change with new releases as we refine the datasets, upstream processing pipeline, and MethBat.
Where possible, we recommend building custom cohort profiles from datasets with identical sample type and processing.

* [cpgIslandExt.sorted.hg38.tsv](./cpgIslandExt.sorted.hg38.tsv) - TSV-reformatted version of the CpG island track from UCSC for hg38 that is essentially a "blank" profile containing only the coordinates of the islands. This is a 5mC / CpG-specific resource. If building a new background / cohort profile, this is the data file to provide when building the initial single-dataset profiles.
* [example_background.tsv](./example_background.tsv) - A small **5mC** cohort background produced with [methbat pileup](../docs/pileup_guide.md) and `methbat build` over the same hg38 CpG island coordinates as [cpgIslandExt.sorted.hg38.tsv](./cpgIslandExt.sorted.hg38.tsv). This example background includes replicates of control samples HG002 and HG005. Rows are keyed by `data_category` (for example population `ALL` plus per-sample labels). It replaces the older `meth_profile_model.tsv` example and is meant for quick tests and documentation-style workflows only; build a custom background from a cohort that matches your study when doing real analyses.
