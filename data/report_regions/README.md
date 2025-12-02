# Report regions
This sub-folder contains region sets that are pre-configured to work with `methbat report`.
We provide brief descriptions of each below, grouped by use case.

## Imprinting regions
The files in this group contain imprinting regions, which are expected to have one allele methylated and the other unmethylated in healthy samples.
Loss of imprinting within these regions is often associated with disease, and they typically manifest as fully methylated or unmethylated in affected samples.
While "AlleleSpecificMethylation" is the expected state for healthy samples, local phasing information may not always be available to confirm the ASM.
In these cases, "neutral" methylation states (e.g., ~50% combined methylation) may also be considered normal.

### `hg38_imprinting_targets.tsv`
* Source: These regions are derived from [Table 2](https://clinicalepigeneticsjournal.biomedcentral.com/articles/10.1186/s13148-022-01358-9/tables/2) of [Mackay et al. 2022](http://doi.org/10.1186/s13148-022-01358-9).
* Coordinates: GRCh38
* Tested with:
  * >200 cell line samples
  * >300 blood sample
  * All samples were WGS with at least 25x mean coverage
* Known problem regions:
  * `H19/IGF2:IG-DMR` - In GRCh38, some versions include ALT contigs that pull reads away from this mapping location, leading to significant loss of alignments and increased classification errors.
  * `MEG3/DLK1:IG-DMRa` - This region is very short and typically only contains 3 CpGs in most datasets. While we did not observe issues in blood samples, we found that cell lines have elevated classification errors in this region.
* Classification summary (blood only) excluding problem regions:
  * PASS - 4,190 (91.52%); detected as proper ASM
  * Inconclusive - 378 (8.26%); could not confirm nor deny ASM
  * AnomalousQcWarning - 8 (0.17%); loss of ASM detected with QC warnings
  * Anomalous - 2 (0.04%); loss of ASM detected without QC warnings

### `hg38_imprinting_autoasm.tsv`
* Source: These regions were created by applying `methbat joint-segment` to >300 WGS blood samples with at least 25x mean coverage. They were then overlapped and labeled with the same identifiers from `imprinting_targets.tsv`. Generally, the coordinates closely match, and we see very marginal differences in the output. However, these may be slightly more accurate regions since they have been derived from HiFi observations.
* Coordinates: GRCh38
* Tested with:
  * >200 cell line samples
  * >300 blood sample
  * All samples were WGS with at least 25x mean coverage
* Known problem regions:
  * `H19/IGF2:IG-DMR` - A consistent ASM region was not identified due to ALT contigs pulling reads away from this mapping location. It has been removed from this set.
  * `MEG3/DLK1:IG-DMRa` - Similar to the base regions, we found that cell lines have elevated classification errors in this region.
* Classification summary (blood only) excluding problem regions:
  * PASS - 4,194 (91.61%); detected as proper ASM
  * Inconclusive - 374 (8.17%); could not confirm nor deny ASM
  * AnomalousQcWarning - 8 (0.17%); loss of ASM detected with QC warnings
  * Anomalous - 2 (0.04%); loss of ASM detected without QC warnings
