# Segmentation guide
This section of the user guide contains information on using `methbat segment` to segment (or split) the genome into regions with similar methylation signatures.
Segmentation clusters the base modifications based on their methylation status into one of three categories: Methylated, Unmethylated, or AlleleSpecificMethylation.
Methylated/Unmethylated is calculated separately from AlleleSpecificMethylation, so there is potentially some overlap in the derived segments.

Table of contents:

* [Segmentation workflow](#segmentation-workflow)
* [Output files](#output-files)

# Segmentation workflow
```bash
methbat segment \
    --input-pileup {PILEUP_BED_GZ} \
    --output-prefix {OUT_PREFIX}
```

Parameters:
* `--input-pileup {PILEUP_BED_GZ}` - the pileup BED produced by [methbat pileup](./pileup_guide.md) (typically `*.{mod}.bed.gz`); this file contains metrics aggregated at each base modification locus
* `--output-prefix {OUT_PREFIX}` - The path to the prefix for all output files.

## Common segmentation options
* `--condense-bed-labels` - Condenses the output labels to an abbreviated form (e.g., Methylated -> M)
* `--enable-haplotype-segmentation` - Enables the output of additional haplotype-specific segments in the output. These segments are generated with a separate segmentation run on each of the individual haplotypes. The output segments will be labeled with a haplotype prefix ("H1" or "H2") followed by the corresponding segmentation category (Methylated or Unmethylated).
* `--enable-nodata-segments` - Enabled the output of additional "NoData" ("ND" if condensed) segments in the output. These segments correspond to regions where combined methylation values are available but the corresponding haplotype data is absent. The output segments will be labeled with a haplotyped prefix ("H1" or "H2"). For example, "H1_NoData" indicated that haplotype 1 is missing in the corresponding region.
* `--strand {combined|forward|reverse}` - Row-level strand filter on the input pileup; default `combined` aggregates across strands. See [pileup output strand semantics](./pileup_guide.md#strand-filter) for what each value selects.

# Output files
Currently, all outputs follow standard BED or BEDGRAPH file formats, the contents of each are defined below:

* `{OUT_PREFIX}.meth_regions.bed` - BED file with merged regions. The fourth column indicates the segment categorization: Methylated (M), Unmethylated (U), or AlleleSpecificMethylation (ASM).
* `{OUT_PREFIX}.combined_methyl.bedgraph` - BEDGRAPH file with values from combined methylation segmentation. The fourth column (`score`) is the segment mean combined methylation written as a percentage.
* `{OUT_PREFIX}.asm.bedgraph` - BEDGRAPH file with values from ASM segmentation. The fourth column (`score`) is the segment mean haplotype methylation difference **(haplotype 2 − haplotype 1)** as a signed percentage (negative when haplotype 1 is more methylated).
* `{OUT_PREFIX}.hap{1/2}.bedgraph` - BEDGRAPH file with values from haplotype-specific segmentation. The fourth column (`score`) is the mean methylation percentage for that haplotype within the segment.

Example (`combined_methyl.bedgraph`; `##` preamble omitted):

```
#chrom	start	end	score
chr1	10468	10641	90.9
chr1	10643	10861	72.6
chr1	10866	11116	87.5
chr1	11117	11842	62.7
```

Example (`asm.bedgraph`; `##` preamble omitted):

```
#chrom	start	end	score
chr1	14485	15749	-5.5
chr1	15768	16869	12.5
chr1	16931	20983	-1.1
chr1	21001	28705	8.3
```
