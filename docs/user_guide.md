# User Guide
Table of contents:

* [Quickstart](#quickstart)
* [Main workflows](#main-workflows)
* [Supported upstream processes](#supported-upstream-processes)
* [Output files](#output-files)
* [Data files](../data/)

# Quickstart
The following command will create a methylation profile for a single dataset:

```bash
methbat profile \
    --input-prefix {IN_PREFIX} \
    --input-regions {IN_REGIONS} \
    --output-region-profile {OUT_PROFILE} \
    --profile-label {LABEL}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus
* `--input-regions {IN_REGIONS}` - the genomic regions of interest; this can be a BED-like coordinate file or a background profile generated from `methbat build ...`; some example profiles are provided in the [data folder](../data/)
* `--output-region-profile {OUT_PROFILE}` - the output profile for this dataset
* `--profile-label {LABEL}` - (optional) if a background profile is provided, this label specified which subset of the background to compare against; defaults to "ALL" and can be specified multiple times

## Quickstart Example
```
methbat profile \
    --input-prefix ./pipeline/cpg_5mc_model/HG001 \
    --input-regions ./data/cpgIslandExt_sorted.tsv \
    --output-region-profile ./output/HG001.meth_region_stats.tsv \
    --profile-label ALL \
    --profile-label FEMALE 

[2023-07-07T15:59:48.961Z INFO  methbat::cli::profile] Input prefix: "./pipeline/cpg_5mc_model/HG001"
[2023-07-07T15:59:48.961Z INFO  methbat::cli::profile] Region profiling:
[2023-07-07T15:59:48.961Z INFO  methbat::cli::profile] 	Input profile regions: "./data/cpgIslandExt_sorted.tsv"
[2023-07-07T15:59:48.961Z INFO  methbat::cli::profile] 	Output profile file: "./output/HG001.meth_region_stats.tsv"
[2023-07-07T15:59:48.961Z INFO  methbat::cli::profile] 	Profile selections: ["ALL", "FEMALE"]
[2023-07-07T15:59:48.961Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_model/HG001.combined.bed"...
[2023-07-07T15:59:48.964Z INFO  methbat::cpg_parser] Model mode auto-detected
[2023-07-07T16:00:17.677Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_model/HG001.hap1.bed"...
[2023-07-07T16:00:40.287Z INFO  methbat::cpg_parser] Loading "./pipeline/cpg_5mc_model/HG001.hap2.bed"...
[2023-07-07T16:01:01.713Z INFO  methbat::cpg_parser] Loading "./data/cpgIslandExt_sorted.tsv"...
[2023-07-07T16:01:01.947Z INFO  methbat::writers::region_writer] Saving region results to "./output/HG001.meth_region_stats.tsv"...
[2023-07-07T16:01:04.403Z INFO  methbat] Process finished successfully.
```

# Main workflows
There are currently three main workflows that are supported by MethBat: 1) [rare methylation analysis](#rare-methylation-analysis), 2) [cohort methylation analysis](#cohort-methylation-analysis), and 3) [segmentation](#segmentation-analysis).

## Cohort analysis workflows
The first two workflows require building a cohort/background methylation profile that can be used to generate statistical comparisons.
The steps for the first two workflows are summarized below, with greater details in the following subsections:

1. Gather a cohort of HiFi datasets that have haplotagged BAM files with methylation information. We recommend using the PacBio best practices pipeline which uses pbmm2, DeepVariant, pbsv, and HiPhase to generate the haplotagged BAM file from a uBAM.
2. Gather individual CpG stats using [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) for each dataset. Both count and model modes are supported, but `methbat` parameters are tuned for model mode.
3. [Generate a basic profile](#generating-a-basic-methylation-profile) for each dataset using `methbat profile ...`. The region file provided in this step will be very similar to a basic BED file.
4. [Generate a cohort methylation profile](#generating-a-cohort-methylation-profile) for the cohort using `methbat build ...`.
5. For [rare methylation analysis](#rare-methylation-analysis), re-run `methbat profile ...` using the cohort methylation profile. This will re-run the same analysis, but will also calculate Z-scores relative to the cohort profile.
6. For [cohort methylation analysis](#cohort-methylation-analysis), run `methbat compare ...` using the cohort methylation profile. The will calculate Z-scores of sub-groups within the population, highlighting regions with different methylation patterns in the sub-groups.

## Generating a basic methylation profile
The following command will create a methylation profile for a single dataset:

```bash
methbat profile \
    --input-prefix {IN_PREFIX} \
    --input-regions {IN_REGIONS} \
    --output-region-profile {OUT_PROFILE}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus; both model and count modes are supported, but the same mode should be used for all datasets; MethBat uses defaults tuned for the model mode
* `--input-regions {IN_REGIONS}` - the genomic regions of interest (e.g. CpG islands); an [example CpG island file](../data/cpgIslandExt.sorted.hg38.tsv) is provided in our [data folder](../data/)
* `--output-region-profile {OUT_PROFILE}` - the [output profile](#single-dataset-profiles) for this dataset

### Example regions file
The baseline regions file is a CSV/TSV containing all of the information in a typical BED file, but with a header.
We have provided the [CpG island file for hg38](../data/cpgIslandExt.sorted.hg38.tsv) with this repo.

Fields: 
* `chrom` - the chromosome of the region
* `start` - the 0-based start of the region, inclusive
* `end` - the 0-based end of the region, exclusive
* `cpg_label` - (optional column) a label assigned to the region

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
By default, MethBat requires Z-scores >= 3.0, absolute deltas >= 0.2, and at least 10 datasets in the background cohort in order to assign any Hypo- or Hyper- states.

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

## Segmentation analysis
Segmentation clusters the CpGs based on their methylation status into one of three categories: Methylated, Unmethylated, or AlleleSpecificMethylation.
Methylated/Unmethylated is calculated separately from AlleleSpecificMethylation, so there is potentially some overlap in the derived segments.

```bash
methbat segment \
    --input-prefix {IN_PREFIX} \
    --output-segments {OUT_BED}
```

Parameters:
* `--input-prefix {IN_PREFIX}` - the prefix for the outputs from [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools), these outputs contain CpG metrics aggregated at each CpG locus
* `--output-segments {OUT_BED}` - The path to the output BED file. The fourth column indicates the segment categorization: Methylated (M), Unmethylated (U), or AlleleSpecificMethylation (ASM)

### Common segmentation options
* `--condense-bed-labels` - Condenses the output labels to an abbreviated form (e.g., Methylated -> M)
* `--enable-haplotype-segmentation` - Enables the output of additional haplotype-specific segments in the output. These segments are generated with a separate segmentation run on each of the individual haplotypes. The output segments will be labeled with a haplotype prefix ("H1" or "H2") followed by the corresponding segmentation category (Methylated or Unmethylated).
* `--enable-nodata-segments` - Enabled the output of additional "NoData" ("ND" if condensed) segments in the output. These segments correspond to regions where combined methylation values are available but the corresponding haplotype data is absent. The output segments will be labeled with a haplotyped prefix ("H1" or "H2"). For example, "H1_NoData" indicated that haplotype 1 is missing in the corresponding region.

# Supported upstream processes
The following upstream processes are supported as inputs to MethBat:

* [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) - both `model` and `count` modes are supported, but default parameters are tuned for `model` mode

# Output files
## Single dataset profiles
The CSV/TSV file containing CpG methylation metrics for the provided regions.
If a background profile is provided, relative metrics (such as Z-scores) will also be generated.

Fields:
* `chrom`, `start`, `end` - the region definition, copied from the input region file
* `summary_label` - a summarization of the methylation status for this region, possible options are below:
  * `NoData` - indicates no CpGs were found inside the region
  * `Uncategorized` - indicates that CpGs were present, but there was not enough evidence to label this region with any of the following labels
  * `Methylated` - indicates that the combined CpG methylation had a high average methylation rate (by default, >=80%)
  * `Unmethylated` - indicates that the combined CpG methylation had a low average methylation rate (by default, <=20%)
  * `AlleleSpecificMethylation` - indicates that a sufficient fraction of the CpGs were phased (by default, >=50%) _and_ that ASM was detected through _both_ a significant Fisher's exact test (by default, p <= 0.01) and difference in mean methylation for haplotypes 1 and 2 (by default, >= 40% methylation delta)
* `compare_label` - a high level summary label of the comparison of this dataset to the background; the "Background comparison" settings control the cut-offs that alter this output, with internally tested heuristics used as defaults; possible options are below:
  * `InsufficientData` - if this dataset does not have CpGs or the the number of datasets in the background is not sufficient
  * `Uncategorized` - if the region does not match any of the summary labels below
  * `HypoASM` - if the region shows significantly _less_ ASM in the dataset than the background; this option has lower priority than the following options
  * `HyperASM` - if the region shows significantly _more_ ASM in the dataset than the background; the `summary_label` must be "AlleleSpecificMethylation"
  * `HypoMethylated` - if the region shows significantly _less_ methylation in the dataset than the background; the `summary_label` must be "Unmethylated"
  * `HyperMethylated` - if the region shows significantly _more_ methylation in the dataset than the background; the `summary_label` must be "Methylated"
* `background_category` - if a background region file was provided, this field will have the label of the compared sub-group; if multiple labels are specified via CLI, there will be one row for each region and label
* `category_pop_count` - the number of datasets in the background population with the same `summary_label`
* `category_pop_freq` - the percentage of datasets (excluding `NoData`) in the background population with the same `summary_label`
* `asm_fishers_pvalue` - the raw p-value from a Fisher's exact test comparing the two haplotypes and the number of reads that are methylated/unmethylated
* `mean_hap{x}_methyl` - the mean (average) methylation ratio for CpGs on an individual haplotype (hap1 and hap2)
* `mean_meth_delta` - the difference in mean methylation ratios between the two haplotypes; `mean_meth_delta = mean_hap2_methyl - mean_hap1_methyl`
* `mean_abs_meth_delta_zscore` - if a background region file was provided, this value is the Z-score comparing `abs(mean_meth_delta)` against the background profile; positive values indicate _more_ evidence of ASM in this dataset relative to the population, negative values indicate _less_
* `mean_combined_methyl` - the mean (average) combined methylation ratio; "combined" here indicates that phasing (i.e. haplotypes) is not considered
* `mean_combined_methyl_zscore` - if a background region file was provided, this value is the Z-score comparing `mean_combined_methyl` against the background profile; positive values indicate this dataset is hyper-methylated relative to the the population, negative values indicate hypo-methylation
* `num_phased_cpgs` - the number of CpGs in the region with haplotagged reads on both haplotypes
* `num_partial_cpgs` - the number of CpGs in the region with haplotagged reads on only one haplotype
* `num_unphased_cpgs` - the number of CpGs in the region with no haplotagged reads

Example:
```
chrom	start	end	summary_label	compare_label	background_category	category_pop_count	category_pop_freq	asm_fishers_pvalue	mean_hap1_methyl	mean_hap2_methyl	mean_meth_delta	mean_abs_meth_delta_zscore	mean_combined_methyl	mean_combined_methyl_zscore	num_phased_cpgs	num_partial_cpgs	num_unphased_cpgs
chr1	28735	29737	Unmethylated	Uncategorized	ALL	75	1.0	0.004919553947553728	0.05885963392250609	0.047044954980294464	-0.01181467894221167	-0.34912373121989750.025268950189202555	-0.1315141967636566	113	0	0
chr1	28735	29737	Unmethylated	Uncategorized	FEMALE	45	1.0	0.004919553947553728	0.05885963392250609	0.047044954980294464	-0.01181467894221167	-0.43383762338663810.025268950189202555	-0.3625714249590449	113	0	0
chr1	135124	135563	Uncategorized	Uncategorized	ALL	11	0.14666666666666667	3.774032618914446e-12	0.9621704619311497	0.7499140499066592	-0.21225641202449116	3.2865300050104986	0.7964857023430671	-0.9735374007607187	32	0	0
chr1	135124	135563	Uncategorized	Uncategorized	FEMALE	6	0.13333333333333333	3.774032618914446e-12	0.9621704619311497	0.7499140499066592	-0.21225641202449116	3.0353565777478027	0.7964857023430671	-0.9943730322724927	32	0	0
...
```

## Cohort/background profiles
The CSV/TSV file containing statistics on a cohort of datasets.
This can also serve as a "background" profile to generate Z-scores relative to the background.
Example cohort background profiles can be found in the [data folder](../data/).

Fields:
* `chrom`, `start`, `end` - the region definition
* `data_category` - the label assigned to the dataset; "ALL" indicates the full cohort; there is one line in the file for each combination of region and `data_category`
* `num_phased` - the number of datasets with `num_phased_cpgs > 0` for this region
* `num_unphased` - the number of datasets with `num_phased_cpgs == 0` for this region
* `NoData` - the number of datasets that have no CpG data for this region; total number of datasets `= num_phased + num_unphased + NoData`
* `Uncategorized`, `Methylated`, `Unmethylated`, `AlleleSpecificMethylation` - the number of datasets with the corresponding `summary_label` for the region
* `avg_abs_meth_deltas`, `stdev_abs_meth_deltas` - the average (mean) and standard deviation of the population's `abs(mean_meth_delta)`; conceptually, this is a measure of average ASM across the cohort, where higher values indicate larger methylation differences between haplotypes 1 and 2; these values are used to calculate `mean_abs_meth_delta_zscore` in a dataset profile
* `avg_combined_methyls`, `stdev_combined_methyls` - the average (mean) and standard deviation of the population's `mean_combined_methyl`; conceptually, this is a measure of average methylated across the cohort, where higher values indicate that the population tends to be methylated, and lower values indicated unmethylated; these values are used to calculate `mean_combined_methyl_zscore` in a dataset profile

Example:
```
chrom	start	end	data_category	num_phased	num_unphased	NoData	Uncategorized	Methylated	Unmethylated	AlleleSpecificMethylation	avg_abs_meth_deltas	stdev_abs_meth_deltas	avg_combined_methyls	stdev_combined_methyls
chr1	28735	29737	ALL	73	2	0	0	0	75	0	0.01838721188173107	0.01882579828232775	0.02586729468921431	0.004549657107262999
chr1	28735	29737	FEMALE	44	1	0	0	0	45	0	0.018232146044057464	0.014792324952708208	0.026786472749753616	0.004185444456143989
chr1	28735	29737	MALE	29	1	0	0	0	30	0	0.01862248418716691	0.023983589672305305	0.024488527598405387	0.004791927756910658
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

Example (this file has been spliced to show different results from FEMALE v. MALE outputs):
```
#chrom	start	end	baseline_category	compare_category	summary_comparison	zscore_avg_abs_meth_deltas	delta_avg_abs_meth_deltas	baseline_num_phased	compare_num_phased	zscore_avg_combined_methyls	delta_avg_combined_methyls	baseline_num_samples	compare_num_samples
chr1	28735	29737	FEMALE	MALE	Uncategorized	0.0783692468300548	0.0003903381431094449	44	29	-2.1384731927805856	-0.0022979451513482282	45	30
chr1	491107	491546	FEMALE	MALE	InsufficientData		0.0	0	0	-1.299378424859551	-0.044848726779527226	16	9
chr1	143326822	143327608	FEMALE	MALE	HyperMethylated	0.6439708246041966	0.09808535591965556	9	7	4.217908672989068	0.2262013390038578	45	30
chrX	19133	20029	FEMALE	MALE	HyperASM	4.961975548625674	0.39109889418176313	27	18	2.7705796060748784	0.16345014506408945	38	29
chrX	9785526	9787061	FEMALE	MALE	HypoMethylated	-2.8538050434741886	-0.10707585326802943	45	9	-11.54167875937393	-0.2214659100846959	45	30
chrX	12791036	12791314	FEMALE	MALE	HypoASM	-6.333953266171334	-0.28676116392809575	45	13	-40.322033758618794	-0.3669797450046681	45	30
...
```
