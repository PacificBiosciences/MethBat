# User Guide
Table of contents:

* [Supported upstream processes](#supported-upstream-processes)
* [Main workflows](#main-workflows)
* [Data files](../data/)

# Supported upstream processes
The following upstream processes are supported as inputs to MethBat:

* [methbat pileup](./pileup_guide.md) - generates per-base-modification pileup BED files (`5mC`, `5hmC`, `6mA`) directly from aligned HiFi BAM files; its output is the input for every other pileup-consuming sub-command

Upgrading from a pre-v1.0.0 workflow that used [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) as the upstream pileup source?
See the [v1.0.0 migration guide](./migration_v1.md) for the full list of CLI and output-format changes.

# Main workflows
There are multiple workflows that are supported by MethBat:
* [Profile-based analyses](./profile_guide.md) - Analyses on a fixed set of pre-defined regions (for example: CpG islands for 5mC)
  * [Rare methylation analysis](./profile_guide.md#rare-methylation-analysis) - Identify regions in a single dataset exhibiting a "rare" methylation patterns relative to a collection of background datasets
  * [Cohort methylation analysis](./profile_guide.md#cohort-methylation-analysis) - Identify regions exhibiting different methylation patterns between case and control datasets
* [Report analysis](./report_guide.md) - Analyze pre-defined regions with known expected methylation patterns (e.g., imprinting regions) and identify anomalous methylation states; requires pre-defined regions with expected methylation categories.
* [Segmentation](./segmentation_guide.md) - Segment (or divide) pileup sites for an individual dataset into regions with a shared methylation pattern; no pre-defined regions required.
* [Joint segmentation](./joint_segmentation_guide.md) - Segment by averaging methylation values across a cohort at each pileup site, then segmenting the averaged values; no pre-defined regions required.
* [Signature generation](./signature_guide.md) - Identify regions exhibiting different methylation patterns between case and control datasets; no pre-defined regions required.
* [Cell type deconvolution](./deconvolution_guide.md) - Estimate cell type proportions from bulk methylation data using a reference atlas; requires a pre-defined cell type atlas.
