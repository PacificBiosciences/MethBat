# v0.14.2
## Fixed
- Fixed an issue where CpG segments with fewer CpGs than the `--min-cpgs` parameter could get created. This primarily occurred with groups of isolated CpGs, and affected `segment`, `joint-segment`, and `signature` modes. These small isolated segments are now filtered after the segmentation step, and therefor removed from all outputs.

# v0.14.1
## Fixed
- Improved error handling for small population sizes when calling `signature`
- Added earlier error checks with better messaging for `signature`

# v0.14.0
## Changes
- Adds a new sub-command `methbat joint-segment`: This sub-command loads all methylation values for a cohort and generates an average methylation for the collection. This average is then segmented into combined methylation and allele-specific methylation segments.
- **Breaking change** - The CLI verbosity flag (`-v`) has been moved into a shared location. Commands that previously used it (e.g. `methbat segment -v ...`) will need to update the position to before the sub-command (e.g. `methbat -v segment ...`).

# v0.13.3
## Changes
- Added support for parsing pb-CpG-tools files that have been compressed (i.e., `{prefix}.combined.bed.gz`) to save storage

# v0.13.2
## Fixed
- Modified the checker for `profile` and `segment` modes to allow for missing hap1 and hap2 BED files. This is most common when pb-CpG-tools is run on unphased datasets. A warning will be produced in the log if these files are missing, and haplotype-specific data will not be generated.

# v0.13.1
## Changes
- Adds two haplotype-specific tracks for `methbat segment`:
  - `{OUT_PREFIX}.hap1.bedgraph` - Contains the underlying methylation values for haplotype 1 for each segmented region
  - `{OUT_PREFIX}.hap2.bedgraph` - Same as above, but for haplotype 2

## Fixed
- Fixed an issue where the BEDGRAPH outputs from `methbat segment` were reporting median values instead of the indicated mean values. This change did not alter the segmentation itself. However, since the value associated with each segment changed, the reported regions in `{OUT_PREFIX}.meth_regions.bed` may be altered to reflect the correct, mean values.

# v0.13.0
## Changes
- Beta release of a new signature mode, `methbat signature`, that will identify regions with different combined methylation in a case-control population. Greater details on usage can be found in the documentation for signature mode.
- User guide documentation has been re-organized into sections based on functionality for easier use

## Fixed
- Fixed an issue where the mean methylation metrics for `methbat profile` were aggregated with unequal weights
- Updated the provided `meth_profile_model.tsv` background file to use the correct equal weights. The changes across most loci are relatively small, with the average delta of ~1% for `avg_abs_meth_deltas` and ~2% for `avg_combined_methyls`.

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
