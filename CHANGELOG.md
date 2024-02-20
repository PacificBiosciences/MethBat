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
