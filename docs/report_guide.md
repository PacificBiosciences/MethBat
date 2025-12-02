# Report guide
This section of the user guide contains information on using `methbat report` to analyze pre-defined regions with known expected methylation patterns, such as imprinting regions.
This approach extracts the CpGs overlapping the pre-defined regions and aggregates the signal for the region, allowing `methbat` to assign labels such as "Methylated" or "AlleleSpecificMethylation" for the regions.
The `methbat report` command then compares the observed methylation patterns against the expected patterns, identifying regions with anomalous methylation states and generating quality control warnings.

Table of contents:

* [Quickstart](#quickstart)
* [Input files](#input-files)
* [Output files](#output-files)

# Quickstart
The following command will create a methylation report for a single dataset:

```bash
methbat report \
    --input-prefix {IN_PREFIX} \
    --input-regions {IN_REGIONS} \
    --output-report {OUT_REPORT}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus
* `--input-regions {IN_REGIONS}` - the genomic regions of interest with expected methylation categories; example region files are provided in the [report_regions data folder](../data/report_regions/) and the format is [specified below](#regions-file)
* `--output-report {OUT_REPORT}` - the output report file (CSV/TSV)

## Quickstart Example
```
methbat report \
    --input-prefix ./pipeline/cpg_5mc_model/HG001 \
    --input-regions ./data/report_regions/imprinting_targets.tsv \
    --output-report ./output/HG001.report.tsv

[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] Input/Output:
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Input prefix: "./pipeline/cpg_5mc_count/HG001"
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Input profile regions: "./data/methbat_report/imprinting_targets.tsv"
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Output report file: "./output/HG001.report.tsv"
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] Labeling heuristics:
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Minimum haplotype coverage: 10
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Minimum ASM phased fraction: 0.75
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Minimum ASM absolute delta mean: 0.5
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Minimum Weak ASM absolute delta mean: 0.3
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Maximum ASM Fishers exact p-value: 0.01
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Maximum unmethylated combined fraction: 0.2
[2025-11-21T19:56:58.295Z INFO  methbat::cli::report] 	Minimum methylated combined fraction: 0.8
[2025-11-21T19:56:58.295Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_count/HG001.combined.bed"...
[2025-11-21T19:57:29.203Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_count/HG001.hap1.bed"...
[2025-11-21T19:57:55.854Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_count/HG001.hap2.bed"...
[2025-11-21T19:58:20.003Z INFO  methbat::reporting] Loading "./data/methbat_report/imprinting_targets.tsv"...
[2025-11-21T19:58:20.035Z INFO  methbat::writers::report_writer] Saving report results to "./output/HG001.report.tsv"...
[2025-11-21T19:58:20.632Z INFO  methbat] Process finished successfully.
```

## Additional options
The following parameters control how methylation categories are assigned to regions:
* `--min-haplotype-coverage {COVERAGE}` - the minimum coverage of a haplotype to consider it "normal" for QC purposes
* `--min-asm-phased-fraction {FRAC}` - the minimum fraction of CpGs in a region that must be phased to consider AlleleSpecificMethylation (ASM)
* `--min-asm-abs-delta-mean {DELTA}` - the minimum absolute difference between mean haplotype methylation fractions to consider ASM
* `--min-weakasm-abs-delta-mean {DELTA}` - the minimum absolute difference between mean haplotype methylation fractions to label a region with the QC flag WeakASM (default: 0.3)
* `--max-asm-fishers-exact {P-VALUE}` - the maximum Fisher's exact test p-value to consider ASM (default: 0.01)
* `--max-unmethylated-combined {FRAC}` - the maximum combined methylation fraction to consider unmethylated status (default: 0.2)
* `--min-methylated-combined {FRAC}` - the minimum combined methylation fraction to consider methylated status (default: 0.8)

# Input files
## Regions file
The regions file is a CSV/TSV containing region coordinates along with expected methylation categories and anomalous categories.
Example region files for imprinting regions are provided in the [report_regions data folder](../data/report_regions/).

Fields: 
* `chrom` - the chromosome of the region
* `start` - the 0-based start of the region, inclusive
* `end` - the 0-based end of the region, exclusive
* `cpg_label` - (optional column) a label assigned to the region
* `expected_category` - the expected methylation category for the region; possible values are: `Uncategorized`, `Methylated`, `Unmethylated`, `AlleleSpecificMethylation`
* `anomalous_categories` - a semicolon-separated list of methylation categories that are considered anomalous for this region; these categories indicate loss of the expected methylation pattern

Example: 
```
chrom	start	end	cpg_label	expected_category	anomalous_categories
chr6	144006940	144008751	PLAGL1:alt-TSS-DMR	AlleleSpecificMethylation	Methylated;Unmethylated
chr7	50781028	50783615	GRB10:alt-TSS-DMR	AlleleSpecificMethylation	Methylated;Unmethylated
chr11	1997581	2003510	H19/IGF2:IG-DMR	AlleleSpecificMethylation	Methylated;Unmethylated
...
```

# Output files
## Report files
The CSV/TSV file containing CpG methylation metrics for the provided regions, along with comparisons against expected methylation patterns.

Fields:
* `chrom`, `start`, `end` - the region definition, copied from the input region file
* `cpg_label` - a pass-through of the optional `cpg_label` field from the input file; empty string if not provided
* `report_summary` - a high level summary label comparing the observed methylation pattern to the expected pattern; possible options are below:
  * `PASS` - the computed category matches the expected category
  * `Inconclusive` - the computed category does not match the expected category or any of the anomalous categories
  * `AnomalousQcWarning` - the computed category matches an anomalous category, but there is a QC warning that should be investigated
  * `Anomalous` - the computed category matches an anomalous category, with no QC warnings
* `qc_warnings` - a semicolon-separated list of quality control warnings; possible values are below:
  * `PASS` - indicates there are no QC warnings
  * `LowPhasedCpGs` - the region has a low number of phased CpGs (below the minimum phased fraction threshold)
  * `LowHaplotypeCoverage` - one or both haplotypes have low coverage (below the minimum haplotype coverage threshold)
  * `WeakASM` - indicates that there is a weak allele-specific methylation signal (for regions where ASM is expected but not detected)
* `expected_category` - the expected methylation category for the region, copied from the input file
* `summary_label` - a summarization of the observed methylation status for this region, possible options are below:
  * `NoData` - indicates no CpGs were found inside the region
  * `Uncategorized` - indicates that CpGs were present, but there was not enough evidence to label this region with any of the following labels
  * `Methylated` - indicates that the combined CpG methylation had a high average methylation rate (by default, >=80%)
  * `Unmethylated` - indicates that the combined CpG methylation had a low average methylation rate (by default, <=20%)
  * `AlleleSpecificMethylation` - indicates that a sufficient fraction of the CpGs were phased (by default, >=75%) _and_ that ASM was detected through _both_ a significant Fisher's exact test (by default, p <= 0.01) and difference in mean methylation for haplotypes 1 and 2 (by default, >= 50% methylation delta)
* `mean_combined_methyl` - the mean (average) combined methylation ratio; "combined" here indicates that phasing (i.e. haplotypes) is not considered
* `mean_meth_delta` - the difference in mean methylation ratios between the two haplotypes; `mean_meth_delta = mean_hap2_methyl - mean_hap1_methyl`
* `mean_hap1_methyl` - the mean (average) methylation ratio for CpGs on haplotype 1
* `mean_hap2_methyl` - the mean (average) methylation ratio for CpGs on haplotype 2
* `asm_fishers_pvalue` - the raw p-value from a Fisher's exact test comparing the two haplotypes and the number of reads that are methylated/unmethylated
* `num_phased_cpgs` - the number of CpGs in the region with haplotagged reads on both haplotypes
* `num_partial_cpgs` - the number of CpGs in the region with haplotagged reads on only one haplotype
* `num_unphased_cpgs` - the number of CpGs in the region with no haplotagged reads
* `median_total_coverage` - the median coverage across all CpGs in the region
* `median_hap1_coverage` - the median coverage for CpGs with haplotype 1 information
* `median_hap2_coverage` - the median coverage for CpGs with haplotype 2 information

Example:
```
chrom	start	end	cpg_label	report_summary	qc_warnings	expected_category	summary_label	mean_combined_methyl	mean_meth_delta	mean_hap1_methyl	mean_hap2_methyl	asm_fishers_pvalue	num_phased_cpgs	num_partial_cpgs	num_unphased_cpgs	median_total_coverage	median_hap1_coverage	median_hap2_coverage
chr6	144006940	144008751	PLAGL1:alt-TSS-DMR	PASS	PASS	AlleleSpecificMethylation	AlleleSpecificMethylation	0.4853123556191952	-0.734328774543706	0.8759114565675648	0.14158268202385849	0.0	143	0	0	32	15	16
...
```

