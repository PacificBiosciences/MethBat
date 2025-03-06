# Joint segmentation guide
This section of the user guide contains information on using `methbat joint-segment` to segment (or split) the genome into regions with similar methylation signatures.
Segmentation clusters the CpGs based on their methylation status into one of three categories: Methylated, Unmethylated, or AlleleSpecificMethylation.
Methylated/Unmethylated is calculated separately from AlleleSpecificMethylation, so there is potentially some overlap in the derived segments.
In contrast to `methbat segment`, this method will average the methylation values from a cohort instead of segmenting an individual dataset.
Thus, this method is best suited for identifying regions with a consistent methylation status across a larger cohort.

Table of contents:

* [Joint segmentation workflow](#joint-segmentation-workflow)
* [Output files](#output-files)

# Joint segmentation workflow
```bash
methbat joint-segment \
    -t {THREADS} \
    --input-collection {COLLECTION} \
    --output-prefix {OUT_PREFIX}
```

Parameters:
* `--threads {THREADS}` - the number of threads to use for both loading CpGs into memory and analyzing putative segments
* `--input-collection {COLLECTION}` - a file defining the cohort; [an example](./signature_guide.md#input-collection-file) is provided in the signature guide
* `--output-prefix {OUT_PREFIX}` - the prefix for all [output signature files](#output-files)

## Common joint segmentation options
* `--condense-bed-labels` - Condenses the output labels to an abbreviated form (e.g., Methylated -> M)
* `--enable-nodata-segments` - Enabled the output of additional "NoData" ("ND" if condensed) segments in the output. These segments correspond to regions where combined methylation values are available but the corresponding haplotype data is absent. The output segments will be labeled with a haplotyped prefix ("H1" or "H2"). For example, "H1_NoData" indicated that haplotype 1 is missing in the corresponding region.

# Output files
Currently, all outputs follow standard BED or BEDGRAPH file formats, the contents of each are defined below:

* `{OUT_PREFIX}.meth_regions.bed` - BED file with merged regions. The fourth column indicates the segment categorization: Methylated (M), Unmethylated (U), or AlleleSpecificMethylation (ASM).
* `{OUT_PREFIX}.combined_methyl.bedgraph` - BEDGRAPH file with raw values from combined methylation segmentation. The fourth column indicates the combined methylation average for the segment; range 0.0 (unmethylated) to 1.0 (methylated).
* `{OUT_PREFIX}.asm.bedgraph` - BEDGRAPH file with raw values from ASM segmentation. The fourth column indicates the ASM average for the segment, calculated as: (max methylation haplotype) - (min methylation haplotype); range 0.0 (no ASM) to 1.0 (absolute ASM).
