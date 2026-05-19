# Migration guide: upgrading to MethBat v1.0.0

This guide is for users upgrading from a pre-v1.0.0 workflow that used [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) to produce the per-site methylation BEDs consumed by MethBat.
Starting in v1.0.0, MethBat produces those BEDs itself via the new [methbat pileup](./pileup_guide.md) sub-command, and every downstream sub-command has been updated to consume the new unified pileup BED format.

See the [v1.0.0 CHANGELOG entry](../CHANGELOG.md) for the high-level list of changes; this file is the detailed before/after reference.

Table of contents:

* [Pileup (pb-CpG-tools -> methbat pileup)](#pileup-pb-cpg-tools---methbat-pileup)
* [Breaking change (methylation percentages)](#breaking-change-methylation-percentages)
* [Per-sub-command CLI changes](#per-sub-command-cli-changes)
  * [build](#build)
  * [compare](#compare)
  * [deconvolve](#deconvolve)
  * [joint-segment](#joint-segment)
  * [profile](#profile)
  * [report](#report)
  * [segment](#segment)
  * [signature](#signature)
* [Output format changes](#output-format-changes)
* [Modification compatibility notes](#modification-compatibility-notes)
* [IGV visualization of pileup](#igv-visualization-of-pileup)

# Pileup (pb-CpG-tools -> methbat pileup)

## Before (pb-CpG-tools count mode)
Users previously ran pb-CpG-tools against a haplotagged BAM to produce three BEDs (combined / hap1 / hap2) that MethBat then consumed by prefix:

```bash
aligned_bam_to_cpg_scores \
    --bam {INPUT_BAM} \
    --output-prefix {OUT_PREFIX} \
    --pileup-mode count \
    --threads {THREADS}
```

Outputs (5mC only; pb-CpG-tools does not produce 5hmC or 6mA pileups):

* `{OUT_PREFIX}.combined.bed.gz` — unphased 5mC pileup (all reads aggregated across both haplotypes)
* `{OUT_PREFIX}.hap1.bed.gz` — 5mC pileup restricted to haplotype 1 reads
* `{OUT_PREFIX}.hap2.bed.gz` — 5mC pileup restricted to haplotype 2 reads

## After (methbat pileup)
`methbat pileup` reads the haplotagged BAM directly and emits a single bgzipped BED per base modification:

```bash
methbat pileup \
    --threads {THREADS} \
    --input-bam {INPUT_BAM} \
    --output-prefix {OUT_PREFIX}
```

Outputs (one file per base modification; each file bundles the unphased and per-haplotype pileups together):

* `{OUT_PREFIX}.5mC.bed.gz` — CpG 5mC pileup (unstranded); unphased + haplotype 1 + haplotype 2 rows
* `{OUT_PREFIX}.5hmC.bed.gz` — 5hmC pileup (stranded); unphased + haplotype 1 + haplotype 2 rows
* `{OUT_PREFIX}.6mA.bed.gz` — 6mA pileup (stranded); unphased + haplotype 1 + haplotype 2 rows
* `{OUT_PREFIX}.summary.json` — run metadata, embedded CLI settings, and per-modification summary statistics (see [Summary JSON](./pileup_guide.md#summary-json) in the pileup guide)

Within each file the three tracks (`Total`, `hap1`, `hap2`) are distinguished by the in-file `type` column; see [Output format changes](#output-format-changes).

Key changes:

* **`aligned_bam_to_cpg_scores`** -> **`methbat pileup`**; no separate upstream tool required.
* **Three files per sample** (`combined` / `hap1` / `hap2`) -> **one file per modification** (`5mC`, `5hmC`, `6mA`), with haplotype tracks distinguished by the in-file `type` column.
* MethBat now produces **5hmC and 6mA** pileups alongside 5mC.
* Pass **`--preset pbcpgtools`** to `methbat pileup` if you need parameter parity with pb-CpG-tools count mode defaults (MAPQ, coverage, and edge trim). This preset is intended for legacy benchmarking only; most users should keep the default settings.

See [the pileup guide](./pileup_guide.md) for the full option reference and a worked example.

# Breaking change (methylation percentages)

This is a **breaking change across many MethBat outputs**: methylation magnitudes that were previously written as raw unit fractions in `[0.0, 1.0]` (or signed haplotype deltas in `[-1.0, 1.0]`) in several TSV and BEDGRAPH columns are now written as **methylation percentages** (for example `82.7` for ~82.7% combined methylation, or `-16.5` for a signed haplotype delta). Any parser, notebook, test fixture, or downstream pipeline that assumed fractional units must be updated.

**Rationale:** long-term consistency with the `mod_score` column in [methbat pileup](./pileup_guide.md) output (already a percentage) and a single convention for methylation signal across commands.

**Affected outputs (non-exhaustive; see per-command guides for column lists):**

| Command / artifact | Typical files | Methylation-related columns now on the percentage scale |
|--------------------|---------------|-------------------------------------------------------------|
| `methbat build` | `*.background.tsv` (cohort profile) | `avg_abs_meth_deltas`, `stdev_abs_meth_deltas`, `avg_combined_methyls`, `stdev_combined_methyls` |
| `methbat profile` | per-dataset region stats TSV | `mean_hap1_methyl`, `mean_hap2_methyl`, `mean_meth_delta`, `mean_combined_methyl`, `mean_combined_methyl_delta` |
| `methbat report` | `*.report.tsv` | `mean_combined_methyl`, `mean_meth_delta`, `mean_hap1_methyl`, `mean_hap2_methyl` |
| `methbat compare` | comparison TSV | `delta_avg_abs_meth_deltas`, `delta_avg_combined_methyls` |
| `methbat signature` | `*.signature_stats.tsv` | `baseline_mean`, `compare_mean`, `baseline_median`, `compare_median` |
| `methbat segment`, `methbat joint-segment` | `*.combined_methyl.bedgraph`, `*.asm.bedgraph`, optional `*.hap{1,2}.bedgraph` | fourth column `score` |

For field-by-field descriptions and updated examples, see the [profile](./profile_guide.md), [report](./report_guide.md), [signature](./signature_guide.md), [segmentation](./segmentation_guide.md), and [joint segmentation](./joint_segmentation_guide.md) guides.

# Per-sub-command CLI changes

All pileup-consuming sub-commands share two common changes:

* The old **`--input-prefix {PREFIX}`** (which pointed at a pb-CpG-tools output prefix and was expanded internally to the three `combined` / `hap1` / `hap2` BEDs) is now **`--input-pileup {BED.GZ}`** and accepts a single unified pileup BED from `methbat pileup` (for example `{OUT_PREFIX}.5mC.bed.gz` for CpG 5mC; the same commands also work with `5hmC` and `6mA` files).
* A new **`--strand {combined|forward|reverse}`** filter is available to restrict analysis to a single strand of the pileup BED. The default is `combined`, which matches the pre-v1.0.0 behavior and is what existing pipelines should use; explicitly specifying `--strand` is only needed for future stranded workflows (e.g. 5hmC / 6mA) that want to split by strand.

Sub-command-specific changes are called out in each section below.

## build

`methbat build` consumes single-dataset profile TSVs produced by `methbat profile`; its **CLI is unchanged** from pre-v1.0.0:

```diff
  methbat build \
      --input-collection {COLLECTION} \
      --output-profile {OUT_PROFILE}
```

**Key change (workflow, not CLI):** because `methbat profile` now reads a different input format, any cohort background profile you previously built from pb-CpG-tools-based profile TSVs will need to be **regenerated** from fresh `methbat profile` outputs if you want it to reflect the new pipeline.

## compare

`methbat compare` consumes the output of `methbat build` and has **no CLI or input-format changes** in this release:

```diff
  methbat compare \
      --input-profile {IN_PROFILE} \
      --output-comparison {OUT_COMPARISON} \
      --baseline-category {BASELINE} \
      --compare-category {COMPARATOR}
```

## deconvolve

```diff
  methbat deconvolve \
-     --input-prefix {PB_CPG_TOOLS_PREFIX} \
+     --input-pileup {OUT_PREFIX}.5mC.bed.gz \
      --atlas-regions {ATLAS_FILE} \
      --output-estimate {OUT_JSON}
```

**Key changes:**

* **`--input-prefix`** -> **`--input-pileup`**.
* Optional new **`--strand`** flag (default `combined`); not needed for the current 5mC deconvolution workflow, but available for future stranded data.
* The cell atlases shipped under [public/data/cell_atlas](../data/cell_atlas/) are 5mC-based; the sub-command has not been validated on `5hmC` / `6mA` pileups. Pair it with the `5mC` file.

## joint-segment

The `joint-segment` default CLI surface is mostly unchanged, but there were some changes to inputs and options:

```diff
  methbat joint-segment \
      --threads {THREADS} \
      --input-collection {COLLECTION} \
      --output-prefix {OUT_PREFIX}
```

**Key changes:**

* The **`filename`** column in the `--input-collection` TSV now points at a single `.bed.gz` produced by `methbat pileup` (for example `/path/to/HG001.5mC.bed.gz`). Previously it was a pb-CpG-tools output prefix. **All rows in the collection must reference files for the same base modification** (e.g. all `5mC` or all `5hmC`).
* Optional new **`--strand`** flag (default `combined`); not needed for existing 5mC cohorts, but available for future stranded data.
* **`--min-cpgs`** renamed to **`--min-sites`** (alias preserved).

## profile

```diff
  methbat profile \
-     --input-prefix {PB_CPG_TOOLS_PREFIX} \
+     --input-pileup {OUT_PREFIX}.5mC.bed.gz \
      --input-regions {IN_REGIONS} \
      --output-region-profile {OUT_PROFILE} \
      --profile-label {LABEL}
```

**Key changes:**

* **`--input-prefix`** -> **`--input-pileup`** (single `.bed.gz` from [methbat pileup](./pileup_guide.md)).
* Optional new **`--strand`** flag (default `combined`); not needed for existing 5mC workflows, but available for future stranded data.
* **`--input-regions`** and **`--output-region-profile`** are now **required** (they were optional in pre-v1.0.0 releases).

## report

```diff
  methbat report \
-     --input-prefix {PB_CPG_TOOLS_PREFIX} \
+     --input-pileup {OUT_PREFIX}.5mC.bed.gz \
      --input-regions {IN_REGIONS} \
      --output-report {OUT_REPORT}
```

**Key changes:**

* **`--input-prefix`** -> **`--input-pileup`**.
* Optional new **`--strand`** flag (default `combined`); not needed for the imprinting / 5mC examples shipped with this release, but available for future stranded data.
* The bundled example regions in [public/data/report_regions](../data/report_regions/) are imprinting targets, which are inherently 5mC / CpG; pair them with the `5mC` pileup BED.

## segment

```diff
  methbat segment \
-     --input-prefix {PB_CPG_TOOLS_PREFIX} \
+     --input-pileup {OUT_PREFIX}.5mC.bed.gz \
      --output-prefix {OUT_PREFIX} \
      --enable-haplotype-segmentation
```

**Key changes:**

* **`--input-prefix`** -> **`--input-pileup`**.
* Optional new **`--strand`** flag (default `combined`); not needed for existing 5mC workflows, but available for future stranded data.
* The segmentation option **`--min-cpgs`** has been renamed to **`--min-sites`**; the old name is still accepted as an alias.

## signature

The `signature` CLI surface has been extended with new sampling options and some input changes:

```diff
  methbat signature \
      --threads {THREADS} \
      --baseline-category {BASELINE} \
      --compare-category {COMPARATOR} \
      --input-collection {COLLECTION} \
      --output-prefix {OUT_PREFIX}
```

The optional parameter `--min-zscore` has been replaced with `--min-welch-t` for clarity.
The legacy option is still accepted as an alias.

**Key changes:**

* The **`filename`** column in the `--input-collection` TSV now points at a single `.bed.gz` produced by `methbat pileup` (same semantics as [joint-segment](#joint-segment)). All rows must be the same base modification.
* Optional new **`--strand`** flag (default `combined`); not needed for existing 5mC cohorts, but available for future stranded data.
* **`--min-cpgs`** renamed to **`--min-sites`** (alias preserved).
* **Significance / BED gating:** Hyper- and hypo-methylated regions in **`signature_regions.bed`** are determined using **Welch’s two-sample \|t\|** against **`--min-welch-t`** (default `5.0`), together with **`--min-delta`** and **`--min-sample-frac`**. The TSV still includes **point-biserial** `r` and `t` columns for reference, but they are **not** used for cutoffs. (Older docs sometimes described the cutoff in terms of a point-biserial *t*-style statistic.)
* **Bootstrap sampling is now off by default.** Previously, bootstrap sampling was always enabled internally. It must now be explicitly enabled with **`--enable-sampling`**. Two companion options **`--num-samples`** (default: 40) and **`--sample-rate`** (default: 0.8) control the sampling behavior when enabled. Users who relied on the previous sampling behavior should add `--enable-sampling` to their invocations.
* **Outputs:** **`signature_stats.tsv`** adds **`summary_label`** and renames several columns. See [Signature stats TSV](#signature-stats-tsv) for a compact old→new column map and [the signature guide](./signature_guide.md#detailed-signature-file) for full field definitions.


# Output format changes

## Unified pileup BED
Each of `{OUT_PREFIX}.5mC.bed.gz`, `{OUT_PREFIX}.5hmC.bed.gz`, and `{OUT_PREFIX}.6mA.bed.gz` is a 15-column TSV (see [pileup_guide.md](./pileup_guide.md#bed-file-format) for the full column reference).
The header order is fixed:

```
#chrom	start	end	name	score	strand	mod_score	type	cov	mod_count	unmod_count	inferred_unmod_count	diff_base_count	avg_mod_score	avg_unmod_score
```

### Preserved from pb-CpG-tools count mode
These columns are identical to their pb-CpG-tools count-mode counterparts (same names, same semantics); their column *positions* shift only because the BED6 fields are now populated in front of them:

* `#chrom` - chromosome of the site
* `start` - 0-based start coordinate (inclusive)
* `end` - 0-based end coordinate (exclusive)
* `mod_score` - methylation percentage with one decimal place (0.0–100.0)
* `type` - haplotype label: `Total` (unphased combined), `hap1`, or `hap2`. All three tracks now live in a **single file** keyed by this column instead of three separate BEDs.
* `cov` - coverage at this site (`mod_count + unmod_count`)
* `mod_count` - number of reads called methylated
* `unmod_count` - number of reads called unmethylated
* `avg_mod_score` - mean modification probability for methylated reads (three decimal places; 0.0–1.0)
* `avg_unmod_score` - mean modification probability for unmethylated reads (three decimal places; 0.0–1.0)

### New in MethBat v1.0.0
Everything below is new in v1.0.0.
BED6-standard fields are now populated so the output parses as a standard BED, and two additional per-site metrics have been added:

* **`name`** — single-letter modification code: `m` (5mC), `h` (5hmC), or `a` (6mA).
* **`score`** — methylation percentage expressed in tenths (0–1000), compatible with typical BED "score" usage.
* **`strand`** — `.` for unstranded 5mC (symmetrical CpG), `+` / `-` for stranded modification types.
* **`inferred_unmod_count`** — reads imputed as unmethylated when pileup coverage from sequenced bases exceeds explicit ML calls at this site. Often 0 for 5mC; typically equal to `unmod_count` for 5hmC and 6mA.
* **`diff_base_count`** — reads whose sequenced base at this coordinate does not match the canonical base for this modification type (e.g. SNPs at a CpG).

## File-layout changes
* One file per modification (`{OUT_PREFIX}.{5mC,5hmC,6mA}.bed.gz`) **instead of one file per haplotype**. The old `combined` / `hap1` / `hap2` split is now an in-file `type` column.
* Each output file ships with a tabix index (`.tbi`).

## Shared `##` comment preamble on every output
All outputs from `methbat pileup` **and** from every downstream sub-command now start with a comment preamble (lines beginning with `##`) that records:

* `##methbat_version` — the methbat version string.
* `##datetime` — UTC timestamp of the run.
* `##command` — the full command line used to produce the file.
* `##base_modification` — `5mC`, `5hmC`, or `6mA` (on pileup outputs and on outputs that are specific to a single modification).
* `##strand` — `combined`, `forward`, or `reverse` on outputs produced with an explicit strand filter.

## Signature stats TSV

`methbat signature` writes **`{OUT_PREFIX}.signature_stats.tsv`** with a shared `##` preamble (same style as other MethBat outputs) followed by a header row. If you have parsers or notebooks built against older column names or semantics, update them as follows.

**New column**

* **`summary_label`** — `HyperMethylated`, `HypoMethylated`, or `Uncategorized`; only the first two labels appear in **`signature_regions.bed`** (fourth BED field).

**Renamed columns** (former name → v1.0.0)

* `z_score` → `welch_t_score`
* `r_value` → `point_biserial_r_value`
* `t_value` → `point_biserial_t_value`
* `num_samples1` → `baseline_num_samples`
* `num_samples2` → `compare_num_samples`
* `mean1` → `baseline_mean`
* `mean2` → `compare_mean`
* `median1` → `baseline_median`
* `median2` → `compare_median`

`chrom`, `start`, and `end` are unchanged. Full column semantics and an example row are in the [signature guide](./signature_guide.md#detailed-signature-file).

## Renames and legacy header fallbacks
* Segmentation-style commands accept **`--min-sites`** in place of **`--min-cpgs`**; the old name is still accepted as an alias.
* Region and profile columns use **`region_label`** in place of the older **`cpg_label`**; MethBat still accepts `cpg_label` as a legacy header when reading inputs, so pre-v1.0.0 region/profile files continue to work.

# Modification compatibility notes
All downstream sub-commands accept `5mC`, `5hmC`, or `6mA` pileup BEDs as input, but their default thresholds, categorization cutoffs, and shipped reference files (atlases, imprinting regions, background profiles) are currently tuned for 5mC.
Running them on `5hmC` or `6mA` pileups is supported, but may require parameter adjustments and regenerated reference inputs to produce meaningful results.

Additionally, **cohort- or atlas-based workflows require consistent modifications**:

* `methbat signature` and `methbat joint-segment` require every row in the `--input-collection` to reference files for the same base modification.
* `methbat deconvolve` has only been tested with 5mC data and 5mC atlases.
* `methbat report` ships with 5mC-specific imprinting region examples under [public/data/report_regions](../data/report_regions/); pair those with the `5mC` pileup BED.

# IGV visualization of pileup
pb-CpG-tools provided additional bigWig files for visualizing pileups in IGV.
This file is not produced by `methbat pileup`.
Instead, we recommend [generating a bedGraph file from the pileup](./pileup_guide.md#igv-visualization-of-pileup).
