# Segmentation guide
This section of the user guide contains information on using `methbat segment` to segment (or split) the genome into regions with similar methylation signatures.
Segmentation clusters the CpGs based on their methylation status into one of three categories: Methylated, Unmethylated, or AlleleSpecificMethylation.
Methylated/Unmethylated is calculated separately from AlleleSpecificMethylation, so there is potentially some overlap in the derived segments.

Table of contents:

* [Segmentation workflow](#segmentation-workflow)
* [Output files](#output-files)

# Segmentation workflow
```bash
methbat segment \
    --input-prefix {IN_PREFIX} \
    --output-segments {OUT_BED}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus
* `--output-prefix {OUT_PREFIX}` - The path to the prefix for all output files.

## Common segmentation options
* `--condense-bed-labels` - Condenses the output labels to an abbreviated form (e.g., Methylated -> M)
* `--enable-haplotype-segmentation` - Enables the output of additional haplotype-specific segments in the output. These segments are generated with a separate segmentation run on each of the individual haplotypes. The output segments will be labeled with a haplotype prefix ("H1" or "H2") followed by the corresponding segmentation category (Methylated or Unmethylated).
* `--enable-nodata-segments` - Enabled the output of additional "NoData" ("ND" if condensed) segments in the output. These segments correspond to regions where combined methylation values are available but the corresponding haplotype data is absent. The output segments will be labeled with a haplotyped prefix ("H1" or "H2"). For example, "H1_NoData" indicated that haplotype 1 is missing in the corresponding region.

# Output files
Currently, all outputs follow standard BED or BEDGRAPH file formats, the contents of each are defined below:

* `{OUT_PREFIX}.meth_regions.bed` - BED file with merged regions. The fourth column indicates the segment categorization: Methylated (M), Unmethylated (U), or AlleleSpecificMethylation (ASM).
* `{OUT_PREFIX}.combined_methyl.bedgraph` - BEDGRAPH file with raw values from combined methylation segmentation. The fourth column indicates the combined methylation average for the segment; range 0.0 (unmethylated) to 1.0 (methylated).
* `{OUT_PREFIX}.asm.bedgraph` - BEDGRAPH file with raw values from ASM segmentation. The fourth column indicates the ASM average for the segment, calculated as: (methylation of haplotype 2) - (methylation of haplotype 1); range -1.0 (H1 methylated, H2 unmethylated) to 1.0 (H1 unmethylated, H2 methylated).
