# Profile guide
This section of the user guide contains information on using `methbat` on pre-defined regions of interest from the genome, such as known CpG islands.
This approach extracts pileup sites overlapping the pre-defined regions and aggregates the signal for the region, allowing `methbat` to assign labels such as "Methylated" or "AlleleSpecificMethylation" for the regions.
The `methbat profile` command is the core of this approach, which will gather these metrics and assign labels for all provided regions.
Additionally, a background cohort can be provided instead of just the regions which allows `methbat` to also assign labels relative to the cohort, such as "HyperMethylated" or "HypoASM".

Table of contents:

* [Quickstart](#quickstart)
* [Profile workflows](#profile-workflows)
* [Output files](#output-files)

# Quickstart
The following command will create a methylation profile for a single dataset:

```bash
methbat profile \
    --input-pileup {PILEUP_BED_GZ} \
    --input-regions {IN_REGIONS} \
    --output-region-profile {OUT_PROFILE} \
    --profile-label {LABEL}
```

Parameters:
* `--input-pileup {PILEUP_BED_GZ}` / `-i` — unified pileup BED from [methbat pileup](./pileup_guide.md); `{OUT_PREFIX}.{mod}.bed.gz` where `{mod}` is `5mC`, `5hmC`, or `6mA`
* `--input-regions {IN_REGIONS}` / `-r` — the genomic regions of interest; this can be a BED-like coordinate file or a background profile generated from `methbat build ...`; some example profiles are provided in the [data folder](../data/)
* `--output-region-profile {OUT_PROFILE}` / `-o` — the output profile for this dataset
* `--strand {combined|forward|reverse}` — (optional) row-level strand filter on the input pileup; default `combined` aggregates across strands. See [pileup output strand semantics](./pileup_guide.md#strand-filter) for what each value selects.
* `--profile-label {LABEL}` — (optional) if a background profile is provided, this label specifies which subset of the background to compare against; defaults to "ALL" and can be specified multiple times

## Quickstart Example
The following command will create a methylation profile for a single dataset:

```
methbat profile \
    --input-pileup ./HG001.5mC.bed.gz \
    --input-regions ./data/cpgIslandExt_sorted.tsv \
    --output-region-profile ./output/HG001.meth_region_stats.tsv
```

Parameters:
* `--input-pileup {PILEUP_BED_GZ}` / `-i` — unified pileup BED from [methbat pileup](./pileup_guide.md); `{OUT_PREFIX}.{mod}.bed.gz` where `{mod}` is one of the supported modifications (`5mC`, `5hmC`, or `6mA`)
* `--input-regions {IN_REGIONS}` / `-r` — the genomic regions of interest (e.g. CpG islands for 5mC workflows); an [example CpG island file](../data/cpgIslandExt.sorted.hg38.tsv) is provided in our [data folder](../data/)
* `--output-region-profile {OUT_PROFILE}` / `-o` — the [output profile](#single-dataset-profiles) for this dataset

# Profile workflows
## Cohort analysis workflows
There are two workflows that require building a cohort/background methylation profile that can be used to generate statistical comparisons.
The steps for the first two workflows are summarized below, with greater details in the following subsections:

1. Gather a cohort of HiFi datasets that have haplotagged BAM files with methylation information. We recommend using the PacBio best practices pipeline which uses pbmm2, DeepVariant, pbsv, and HiPhase to generate the haplotagged BAM file from a uBAM.
2. Gather unified pileup outputs using [methbat pileup](./pileup_guide.md) for each dataset. See [methbat pileup](./pileup_guide.md) for details.
3. [Generate a basic profile](#generating-a-basic-methylation-profile) for each dataset using `methbat profile ...`. The region file provided in this step will be very similar to a basic BED file.
4. [Generate a cohort methylation profile](#generating-a-cohort-methylation-profile) for the cohort using `methbat build ...`.
5. For [rare methylation analysis](#rare-methylation-analysis), re-run `methbat profile ...` using the cohort methylation profile. This will re-run the same analysis, but will also calculate Z-scores relative to the cohort profile.
6. For [cohort methylation analysis](#cohort-methylation-analysis), run `methbat compare ...` using the cohort methylation profile. The will calculate Z-scores of sub-groups within the population, highlighting regions with different methylation patterns in the sub-groups.

## Example regions file
The baseline regions file is a CSV/TSV containing all of the information in a typical BED file, but with a header.
We have provided the [CpG island file for hg38](../data/cpgIslandExt.sorted.hg38.tsv) with this repo.

Fields: 
* `chrom` - the chromosome of the region
* `start` - the 0-based start of the region, inclusive
* `end` - the 0-based end of the region, exclusive
* `region_label` - (optional column) a label assigned to the region; legacy header `cpg_label` is accepted when reading

Example: 
```
chrom	start	end	cpg_label
chr1	28735	29737	CpG:_111
chr1	135124	135563	CpG:_30
chr1	199251	200121	CpG:_104
chr1	368792	370063	CpG:_99
chr1	381172	382185	CpG:_84
chr1	491107	491546	CpG:_29
chr1	597839	598734	CpG:_94
chr1	609358	611269	CpG:_171
chr1	778604	779167	CpG:_60
...
```

## Generating a cohort methylation profile
The following command will create a cohort methylation profile:

```bash
methbat build \
    --input-collection {IN_COHORT_FILE} \
    --output-profile {OUT_PROFILE}
```

Parameters:
* `--input-collection {IN_COHORT_FILE}` - a file defining the cohort; an example is shown below
* `--output-profile {OUT_PROFILE}` - the [output cohort profile](#cohortbackground-profiles)

### Example cohort file
The cohort file provides information to MethBat regarding the identifier, location (file path), and any associated labels for each dataset that will be a part of the cohort.

Fields:
* `identifier` - a unique string identifier for the dataset
* `filename` - the file path to the profile for this dataset (output of `methbat profile --output-region-profile ...`)
* `labels` - any labels associated with the dataset; all datasets will always go into the "ALL" category; this field can be blank or have multiple labels that are ';'-delimited; for our example below, HG005 has both the "MALE" and "AFFECTED" labels

Example:
```
identifier	filename	labels
HG001	/path/to/HG001.profile.tsv	FEMALE
HG002	/path/to/HG002.profile.tsv	
HG005	/path/to/HG005.profile.tsv	MALE;AFFECTED
...
```

## Rare methylation analysis
If a full background / cohort profile is provided, MethBat will automatically annotate regions with unusual methylation states in the `compare_label` of the [output single dataset profile](#single-dataset-profiles).
These cutoffs used to define unusual can all be controlled with parameters under the "Background comparison" group in the CLI.
By default, MethBat requires Z-scores >= 3.0, absolute methylation deltas >= 20%, and at least 10 datasets in the background cohort in order to assign any Hypo- or Hyper- states.

## Cohort methylation analysis
This step will compare two categories in a cohort against each other and label regions with significant differences.
For this approach, one category must be specified as the "baseline" and the other as the "comparator".

```bash
methbat compare \
    --input-profile {IN_COHORT_PROFILE} \
    --output-comparison {OUT_COMPARISON} \
    --baseline-category {BASELINE} \
    --compare-category {COMPARE}
```

Parameters:
* `--input-profile {IN_COHORT_PROFILE}` - the cohort profile from `methbat build ...`
* `--output-comparison {OUT_COMPARISON}` - the output CSV/TSV [comparison file](#cohort-comparison-files)
* `--baseline-category {BASELINE}` - the baseline category, all stats will be relative to the datasets in this category
* `--compare-category {COMPARE}` - the comparator category

# Output files
## Single dataset profiles
The CSV/TSV file containing region-level methylation metrics for the provided regions.
If a background profile is provided, relative metrics (such as Z-scores) will also be generated.
The file begins with comment lines (`##`) for methbat version, run time, and command line (see shared header behavior in [pileup output](./pileup_guide.md#output-files)), then a single column header row and data rows.

Fields:
* `chrom`, `start`, `end` - the region definition, copied from the input region file
* `region_label` - a pass-through of the optional label from the background file; empty string if not provided (legacy header `cpg_label` is accepted when reading inputs)
* `summary_label` - a summarization of the methylation status for this region, possible options are below:
  * `NoData` - indicates no pileup sites were found inside the region
  * `Uncategorized` - indicates that sites were present, but there was not enough evidence to label this region with any of the following labels
  * `Methylated` - indicates that the combined methylation had a high average methylation rate (by default, >=80%)
  * `Unmethylated` - indicates that the combined methylation had a low average methylation rate (by default, <=20%)
  * `AlleleSpecificMethylation` - indicates that a sufficient fraction of sites were phased (by default, >=50%) _and_ that ASM was detected through _both_ a significant Fisher's exact test (by default, p <= 0.01) and difference in mean haplotype methylation (by default, >= 50% methylation delta)
* `compare_label` - a high level summary label of the comparison of this dataset to the background; the "Background comparison" settings control the cut-offs that alter this output, with internally tested heuristics used as defaults; possible options are below:
  * `InsufficientData` - if this dataset does not have usable pileup data or the number of datasets in the background is not sufficient
  * `Uncategorized` - if the region does not match any of the summary labels below
  * `HypoASM` - if the region shows significantly _less_ ASM in the dataset than the background; this option has lower priority than the following options
  * `HyperASM` - if the region shows significantly _more_ ASM in the dataset than the background; the `summary_label` must be "AlleleSpecificMethylation"
  * `HypoMethylated` - if the region shows significantly _less_ methylation in the dataset than the background; the `summary_label` must be "Unmethylated"
  * `HyperMethylated` - if the region shows significantly _more_ methylation in the dataset than the background; the `summary_label` must be "Methylated"
* `background_category` - if a background region file was provided, this field will have the label of the compared sub-group; if multiple labels are specified via CLI, there will be one row for each region and label
* `category_pop_count` - the number of datasets in the background population with the same `summary_label`
* `category_pop_freq` - the percentage of datasets (excluding `NoData`) in the background population with the same `summary_label`
* `asm_fishers_pvalue` - the raw p-value from a Fisher's exact test comparing the two haplotypes and the number of reads that are methylated/unmethylated
* `mean_hap{x}_methyl` - the mean (average) haplotype methylation percentage on haplotype 1 or 2
* `mean_meth_delta` - the difference in mean haplotype methylation percentages; `mean_meth_delta = mean_hap2_methyl - mean_hap1_methyl`
* `mean_abs_meth_delta_zscore` - if a background region file was provided, this value is the Z-score comparing `abs(mean_meth_delta)` against the background profile; positive values indicate _more_ evidence of ASM in this dataset relative to the population, negative values indicate _less_
* `mean_combined_methyl` - the mean (average) combined methylation percentage; "combined" here indicates that phasing (i.e. haplotypes) is not considered
* `mean_combined_methyl_delta` - if a background region file was provided, this value is the raw delta comparing the sample to the population mean combined methylation: `mean_combined_methyl - population_mean_combined_methyl`; positive values indicate this dataset is hyper-methylated relative to the population, negative values indicate hypo-methylation
* `mean_combined_methyl_zscore` - if a background region file was provided, this value is the corresponding Z-score for `mean_combined_methyl_delta`
* `num_phased_sites` - the number of pileup sites in the region with haplotagged reads on both haplotypes
* `num_partial_sites` - the number of pileup sites in the region with haplotagged reads on only one haplotype
* `num_unphased_sites` - the number of pileup sites in the region with no haplotagged reads
* `median_total_coverage` - the median coverage across all sites in the region
* `median_hap1_coverage` - the median coverage for sites with haplotype 1 information
* `median_hap2_coverage` - the median coverage for sites with haplotype 2 information

Example:
```
chrom	start	end	region_label	summary_label	compare_label	background_category	category_pop_count	category_pop_freq	asm_fishers_pvalue	mean_hap1_methyl	mean_hap2_methyl	mean_meth_delta	mean_abs_meth_delta_zscore	mean_combined_methyl	mean_combined_methyl_delta	mean_combined_methyl_zscore	num_phased_sites	num_partial_sites	num_unphased_sites	median_total_coverage	median_hap1_coverage	median_hap2_coverage
chr1	28735	29737	CpG:_111	Unmethylated	Uncategorized	ALL	10	1.000	5.472211e-2	2.5	1.3	-1.3	-0.731	4.9	-1.0	-0.279	114	0	0	22	8	7
chr1	135124	135563	CpG:_30	Uncategorized	Uncategorized	ALL	6	0.600	9.346681e-1	78.5	79.0	0.4	-1.123	78.2	-1.9	-0.323	30	0	0	31	17	13
chr1	199251	200121	CpG:_104	Unmethylated	Uncategorized	ALL	10	1.000	1.000000e0	6.0	5.9	-0.1	-1.286	5.6	-1.9	-0.498	105	0	0	22	12	7
chr1	368792	370063	CpG:_99	NoData	InsufficientData	ALL			1.000000e0								0	0	0	0	0	0
chr1	381172	382185	CpG:_84	Methylated	Uncategorized	ALL	6	0.857	1.000000e0					95.8	3.3	0.444	0	0	83	4	0	0
chr1	491107	491546	CpG:_29	Methylated	Uncategorized	ALL	5	0.556	1.000000e0					82.8	2.5	0.364	0	0	30	6	0	0
...
```

## Cohort/background profiles
The CSV/TSV file containing statistics on a cohort of datasets.
This can also serve as a "background" profile to generate Z-scores relative to the background.
Example cohort background profiles can be found in the [data folder](../data/).

Fields:
* `chrom`, `start`, `end` - the region definition
* `region_label` - a pass-through of the optional label from the profile files; empty string if not provided (legacy header `cpg_label` is accepted when reading)
* `data_category` - the label assigned to the dataset; "ALL" indicates the full cohort; there is one line in the file for each combination of region and `data_category`
* `num_phased` - the number of datasets with `num_phased_sites > 0` for this region
* `num_unphased` - the number of datasets with `num_phased_sites == 0` for this region
* `NoData` - the number of datasets that have no pileup data for this region; total number of datasets `= num_phased + num_unphased + NoData`
* `Uncategorized`, `Methylated`, `Unmethylated`, `AlleleSpecificMethylation` - the number of datasets with the corresponding `summary_label` for the region
* `avg_abs_meth_deltas`, `stdev_abs_meth_deltas` - the average (mean) and standard deviation of the population's `abs(mean_meth_delta)`; conceptually, this is a measure of average ASM across the cohort, where higher values indicate larger methylation differences between haplotypes 1 and 2; these values are used to calculate `mean_abs_meth_delta_zscore` in a dataset profile
* `avg_combined_methyls`, `stdev_combined_methyls` - the average (mean) and standard deviation of the population's `mean_combined_methyl`; conceptually, this is a measure of average methylated across the cohort, where higher values indicate that the population tends to be methylated, and lower values indicated unmethylated; these values are used to calculate `mean_combined_methyl_zscore` in a dataset profile

Example:
```
chrom	start	end	region_label	data_category	num_phased	num_unphased	NoData	Uncategorized	Methylated	Unmethylated	AlleleSpecificMethylation	avg_abs_meth_deltas	stdev_abs_meth_deltas	avg_combined_methyls	stdev_combined_methyls
chr1	28735	29737	CpG:_111	ALL	5	5	0	0	0	10	0	2.0	1.0	5.9	3.5
chr1	28735	29737	CpG:_111	HG002	3	3	0	0	0	6	0	1.8	0.6	7.1	4.2
chr1	28735	29737	CpG:_111	HG005	2	2	0	0	0	4	0	2.2	1.7	4.1	0.5
chr1	135124	135563	CpG:_30	ALL	10	0	0	6	4	0	0	4.6	3.7	80.1	5.8
chr1	135124	135563	CpG:_30	HG002	6	0	0	4	2	0	0	2.9	2.8	80.6	7.6
chr1	135124	135563	CpG:_30	HG005	4	0	0	2	2	0	0	7.3	3.4	79.2	1.8
...
```

## Rare event results
The output when providing a background file will be identical to the [single dataset profiles](#single-dataset-profiles).
For further filtering in a rare disease context, we recommend assigning gene labels to each CpG island (e.g., genes with +-10kbp) and further filtering by genes associated with known phenotypes.
Anecdotally, this tends to leave fewer than 20 CpG islands with aberannt Hypo-/Hyper- states to review.

## Cohort comparison files
The CSV/TSV file containing statistics on different sub-groups within the full cohort (e.g. case v. control).

Fields:
* `chrom`, `start`, `end` - the region definition
* `baseline_category`, `compare_category` - the baseline and comparator categories
* `summary_comparison` - a high level summary label of the comparison result; the "Comparison thresholds" settings control the cut-offs that alter this output, with internally tested heuristics used as defaults; possible options are below:
  * `InsufficientData` - if the number of datasets in each category is not sufficient for comparison (default: 10)
  * `Uncategorized` - if the region does not match any of the summary labels below
  * `HypoASM` - if the region shows significantly _less_ ASM in the comparator than the baseline
  * `HyperASM` - if the region shows significantly _more_ ASM in the comparator than the baseline
  * `HypoMethylated` - if the region shows significantly _less_ methylation in the comparator than the baseline
  * `HyperMethylated` - if the region shows significantly _more_ methylation in the comparator than the baseline
* `zscore_avg_abs_meth_deltas`, `delta_avg_abs_meth_deltas` - the Z-score and raw delta of the compare v. baseline for average absolute methylation delta; if the Z-score is sufficiently low, it will be flagged as `HypoASM`; if the Z-score is sufficiently high, it will be flagged as `HyperASM`
* `baseline_num_phased`, `compare_num_phased` - the number of _phased_ datasets in baseline and compare for this region; if this is not sufficiently high, the region will not be analyzed for ASM deviations
* `zscore_avg_combined_methyls`, `delta_avg_combined_methyls` - the Z-score and raw delta of the compare v. baseline for average combined methylation; if the Z-score is sufficiently low, it will be flagged as `HypoMethylated`; if the Z-score is sufficiently high, it will be flagged as `HyperMethylated`
* `baseline_num_samples`, `compare_num_samples` - the number of datasets (both phased and unphased) in baseline and compare for this region; if this is not sufficiently high, the region will be flagged as `InsufficientData`

Example:
```
chrom	start	end	baseline_category	compare_category	summary_comparison	zscore_avg_abs_meth_deltas	delta_avg_abs_meth_deltas	baseline_num_phased	compare_num_phased	zscore_avg_combined_methyls	delta_avg_combined_methyls	baseline_num_samples	compare_num_samples
chr1	28735	29737	HG002	HG005	Uncategorized	0.320	0.4	3	2	-1.731	-3.0	6	4
chr1	135124	135563	HG002	HG005	Uncategorized	2.148	4.4	6	4	-0.433	-1.4	6	4
chr1	199251	200121	HG002	HG005	Uncategorized	0.565	0.4	5	3	-2.533	-4.5	6	4
chr1	368792	370063	HG002	HG005	InsufficientData		0.0	0	0	-6.765	-20.7	1	3
chr1	381172	382185	HG002	HG005	InsufficientData		0.0	0	0	2.659	8.8	5	2
chr1	491107	491546	HG002	HG005	Uncategorized		8.4	0	2	-0.861	-3.6	5	4
...
```
