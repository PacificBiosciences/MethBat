# Joint segmentation guide
This section of the user guide contains information on using `methbat joint-segment` to segment (or split) the genome into regions with similar methylation signatures.
Segmentation clusters pileup sites based on their methylation status into one of three categories: Methylated, Unmethylated, or AlleleSpecificMethylation.
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
* `--threads {THREADS}` - the number of threads to use for both loading pileup sites into memory and analyzing putative segments
* `--input-collection {COLLECTION}` - a file defining the cohort; [an example](./signature_guide.md#input-collection-file) is provided in the signature guide
* `--output-prefix {OUT_PREFIX}` - the prefix for all [joint-segment output files](#output-files)

## Common joint segmentation options
* `--condense-bed-labels` - Condenses the output labels to an abbreviated form (e.g., Methylated -> M)
* `--enable-nodata-segments` - Enabled the output of additional "NoData" ("ND" if condensed) segments in the output. These segments correspond to regions where combined methylation values are available but the corresponding haplotype data is absent. The output segments will be labeled with a haplotyped prefix ("H1" or "H2"). For example, "H1_NoData" indicated that haplotype 1 is missing in the corresponding region.
* `--strand {combined|forward|reverse}` - Row-level strand filter on every per-sample pileup loaded from the collection; default `combined` aggregates across strands. See [pileup output strand semantics](./pileup_guide.md#strand-filter) for what each value selects.

# Output files
Currently, all outputs follow standard BED or BEDGRAPH file formats, the contents of each are defined below:

* `{OUT_PREFIX}.meth_regions.bed` - BED file with merged regions. The fourth column indicates the segment categorization: Methylated (M), Unmethylated (U), or AlleleSpecificMethylation (ASM).
* `{OUT_PREFIX}.combined_methyl.bedgraph` - BEDGRAPH file with cohort-averaged combined methylation segmentation. The fourth column (`score`) is the segment mean combined methylation as a percentage.
* `{OUT_PREFIX}.asm.bedgraph` - BEDGRAPH file with cohort-averaged ASM segmentation. The fourth column (`score`) is the spread between haplotypes **(max mean methylation − min mean methylation)** over the cohort-averaged tracks, written as a non-negative percentage (0 means no spread).

Example (`combined_methyl.bedgraph`; `##` preamble omitted):

```
#chrom	start	end	score
chr1	10678	10926	76.1
chr1	10928	11163	88.3
chr1	11183	14435	66.8
chr1	14468	20590	87.1
```

Example (`asm.bedgraph`; `##` preamble omitted):

```
#chrom	start	end	score
chr1	15379	17452	13.7
chr1	17477	18152	7.9
chr1	18212	20784	12.6
chr1	20966	24412	17.8
```
