# User Guide
Table of contents:

* [Main workflows](#main-workflows)
* [Supported upstream processes](#supported-upstream-processes)
* [Data files](../data/)

# Main workflows
There are currently seven main workflows that are supported by MethBat:
* [Rare methylation analysis](./profile_guide.md#rare-methylation-analysis) - Identify regions in a single dataset exhibiting a "rare" methylation patterns relative to a collection of background datasets; requires pre-defined regions such as all known CpG islands.
*  [Cohort methylation analysis](./profile_guide.md#cohort-methylation-analysis) - Identify regions exhibiting different methylation patterns between case and control datasets; requires pre-defined regions such as all known CpG islands.
*  [Report analysis](./report_guide.md) - Analyze pre-defined regions with known expected methylation patterns (e.g., imprinting regions) and identify anomalous methylation states; requires pre-defined regions with expected methylation categories.
*  [Segmentation](./segmentation_guide.md) - Segment (or divide) CpGs for an individual dataset into regions with a shared methylation pattern; no pre-defined regions required.
*  [Joint segmentation](./joint_segmentation_guide.md) - Segment CpGs by averaging methylation values across a cohort and then segmenting the averaged values; no pre-defined regions required.
*  [Signature generation](./signature_guide.md) - Identify regions exhibiting different methylation patterns between case and control datasets; no pre-defined regions required.
*  [Cell type deconvolution](./deconvolution_guide.md) - Estimate cell type proportions from bulk methylation data using a reference atlas; requires a pre-defined cell type atlas.

# Supported upstream processes
The following upstream processes are supported as inputs to MethBat:

* [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) - both `model` and `count` modes are supported, but default parameters are tuned for `model` mode
