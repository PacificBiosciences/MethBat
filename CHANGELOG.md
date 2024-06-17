# v0.12.0
## Changes
- **Breaking change**: The `methbat segment` output `--output-segments` has been changed to `--output-prefix`. It will now generate the following output files with extensions of the given prefix:
  - `{OUT_PREFIX}.meth_regions.bed` - The BED file with merged regions, same output as the previous `--output-segments` option
  - `{OUT_PREFIX}.combined_methyl.bedgraph` - _New_ output that contains the underlying combined methylation values for each segmented region
  - `{OUT_PREFIX}.asm.bedgraph` - _New_ output that contains the underlying ASM values for each segmented region

# v0.11.0
## Changes
- Modifies the segmentation algorithm to prevent large segments from spanning CpG data gaps. The new algorithm will automatically create segment break points between two consecutive CpGs that exceed a maximum gap threshold. This reduces segment size inflation around regions with low coverage and/or absent phasing information.
- Adds a CLI option `--max-gap` to control the maximum gap threshold, default: 1000 bp.

# v0.10.1
## Fixed
- Fixed a panic caused by low sample sizes in the `compare` command. These will now be flagged as `InsufficientData`.

# v0.10.0
## Changes
- Adds a CLI option `--enable-haplotype-segmentation` that will run segmentation on the individual haplotypes (H1 and H2) and save those segments in the output BED file. For example, an unmethylated region on haplotype 2 will have the label "H2_Unmethylated" (or "H2_U" if condensed).
- Adds a CLI option `--enable-nodata-segments` that will generate segments where haplotype data is missing. These will be flagged like "H1_NoData" (or "H1_ND" if condensed).

# v0.9.0
## Changes
- Adds a new segmentation mode based on Circular Binary Segmentation. This mode generates a BED file of methylated, unmethylated, and ASM regions. The mode is executed with `methbat segment ...`, see README for more details.

## Fixed
- Improved error handling for all CLI interfaces

# v0.8.3
## Changes
- Initial alpha release
- Includes initial version of HPRC background profiles
