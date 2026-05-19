# Pileup guide
This section of the user guide describes **`methbat pileup`**, which generates per-base-modification pileup BED files directly from aligned BAM files.
The subcommand reads base modification tags (MM/ML) from HiFi BAM files and aggregates methylation calls at each reported site (for example CG context for 5mC, or stranded loci for 5hmC and 6mA).
The output is one bgzipped BED per base modification type (`5mC`, `5hmC`, `6mA`); combined (unphased) and per-haplotype rows share each file and are distinguished by the `type` column (`Total`, `hap1`, `hap2`).
Each run also writes a summary file with run metadata and per-methylation-type summary statistics.
A **`pbcpgtools`** preset is available for workflows that need parity with [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) **count mode** defaults (see [Presets](#pileup-options)).

Table of contents:

* [Quickstart](#quickstart)
* [Pileup options](#pileup-options)
* [Output files](#output-files)
* [Downstream sub-commands](#downstream-sub-commands)
* [IGV visualization of pileup](#igv-visualization-of-pileup)

# Quickstart
The following command will generate methylation pileup files from a BAM file:

```bash
methbat pileup \
    --threads {THREADS} \
    --input-bam {INPUT_BAM} \
    --output-prefix {OUT_PREFIX}
```

Required parameters:

* `--input-bam {INPUT_BAM}` — aligned BAM with MM/ML modification tags; must be indexed (`.bai`); may be repeated to merge pileups across BAM files
* `--output-prefix {OUT_PREFIX}` — prefix for all output pileup files

Optional parameters are described in [Pileup options](#pileup-options). For a full list, run `methbat pileup -h`.

# Pileup options

| Option | Default | Description |
|--------|---------|-------------|
| `--threads` / `-t` | `1` | Parallelism for processing genomic regions |
| `--preset` / `-p` | (none) | Preset configuration; see below |
| `--ignore-read-groups` | off | Allow input BAMs with different sample names (otherwise methbat errors) |
| `--min-mapq` | `1` | Minimum mapping quality for a read to be counted |
| `--min-coverage` | `4` | Minimum reads with modification information required to emit a site |
| `--edge-trimming-size` | `20` | Bases trimmed from each read end for coverage and methylation |
| `--include-empty-sites` | off | For 6mA, also emit sites with no methylated reads |

## Presets

**Most users should use the default settings provided by MethBat.**
However, the following presets are provided in addition to the defaults:

* `pbcpgtools` — **Intended only for legacy compatibility**, matching the behavior of [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools) count mode. This alters settings on the CLI to match those of pb-CpG-tools, and also allows reads with empty methylation tags to contribute to the pileup, which is _not_ recommended for recent datasets.

# Output files
The pileup generates one bgzipped BED per modification family and a tabix index (`.tbi`) for each.
Use these files as inputs to downstream tools such as `methbat profile` via `--input-pileup` (point at the appropriate `{OUT_PREFIX}.{mod}.bed.gz` for your modification of interest).

Output files:

* `{OUT_PREFIX}.5mC.bed.gz` — 5mC CpG pileup (all tracks: `type` = `Total`, `hap1`, or `hap2`)
* `{OUT_PREFIX}.5hmC.bed.gz` — 5hmC pileup (forward and reverse strand rows in the same file)
* `{OUT_PREFIX}.6mA.bed.gz` — 6mA pileup (forward and reverse strand rows in the same file)
* `{OUT_PREFIX}.summary.json` — summary statistics for the run (see [Summary JSON](#summary-json))

Each BED file begins with comment lines (`##`) describing the methbat version, run time, command line, and base modification type, followed by a single header row and data rows (see [BED file format](#bed-file-format)).

## BED file format
Each BED file is tab-separated. After the `##` comment block, the first non-comment line is the column header; every data row has **15** fields: BED6 (`#chrom` through `strand`) plus nine pileup-specific columns.

**Comment preamble** (each line starts with `##`):

* `methbat_version` — methbat version string
* `datetime` — UTC timestamp of the run
* `command` — full command line
* `base_modification` — `5mC`, `5hmC`, or `6mA` for this file

**BED6 columns**

* `#chrom` — chromosome name
* `start` — 0-based start of the modification coordinate (inclusive)
* `end` — 0-based end (exclusive); always `start + 1` for pileup rows
* `name` — single-letter modification code: `m` (5mC), `h` (5hmC), or `a` (6mA)
* `score` — methylation percentage expressed in **tenths** (0–1000), using the same rounding as `mod_score` (compatible with typical BED “score” usage); example: if `mod_score` = 97.2, then `score` = 972
* `strand` — `.` for unstranded 5mC (symmetrical CpG), `+` (forward) or `-` (reverse) for stranded modification types

**Extended columns**

* `mod_score` — methylation percentage with one decimal place; range 0.0–100.0
* `type` — pileup track: `Total` (unphased), `hap1`, or `hap2`
* `cov` — total coverage at this site (`mod_count + unmod_count`)
* `mod_count` — reads called methylated
* `unmod_count` — reads called unmethylated
* `inferred_unmod_count` — reads imputed as unmethylated when pileup coverage from sequenced bases exceeds explicit ML calls at this site; often 0 for 5mC, typically equal to `unmod_count` for 5hmC and 6mA
* `diff_base_count` — reads whose sequenced base at this coordinate does not match the canonical base for this modification type (e.g. SNPs at a CpG)
* `avg_mod_score` — average modification probability for methylated reads (three decimal places; 0.0–1.0)
* `avg_unmod_score` — average modification probability for unmethylated reads (three decimal places; 0.0–1.0)

**Compatibility with pb-CpG-tools count output**

pb-CpG-tools count-style BED omits the middle BED fields and the two methbat-specific count columns; methbat’s layout is a strict superset so that `mod_count` and `unmod_count` remain in the same logical positions for parsers that expect the extended BED layout.

Example (data rows only; your file will include `##` lines above the header):

```
#chrom	start	end	name	score	strand	mod_score	type	cov	mod_count	unmod_count	inferred_unmod_count	diff_base_count	avg_mod_score	avg_unmod_score
chr1  28735  28736  m  33   .  3.3   Total  30  1   29  0  0  0.783  0.050
chr1  28737  28738  m  0    .  0.0   Total  30  0   30  0  0  0.000  0.045
chr1  28760  28761  m  967  .  96.7  Total  30  29  1   0  0  0.910  0.012
...
```

Example with stranded modification (`name` / `strand` differ from 5mC):

```
#chrom	start	end	name	score	strand	mod_score	type	cov	mod_count	unmod_count	inferred_unmod_count	diff_base_count	avg_mod_score	avg_unmod_score
chr22  100200  100201  h  333  +  33.3  hap2  3  1  2  0  0  0.900  0.100
...
```

## Summary JSON
Alongside the BED files, `methbat pileup` always writes a JSON summary at `{OUT_PREFIX}.summary.json` containing run metadata, the CLI settings that were used, and per-methylation-type statistics.

Top-level keys:

* `version` — methbat version string (same as the `##methbat_version` header in the BED files)
* `datetime` — UTC timestamp of the run (same as the `##datetime` header)
* `command` — full command line invoked (same as the `##command` header)
* `cli_settings` — serialized pileup CLI options (`input_bams`, `output_prefix`, thresholds, preset, etc.)
* `statistics` — an object keyed by merged modification type (`"5mC"`, `"5hmC"`, `"6mA"`); stranded summary metrics are not reported

For each methylation type the statistics aggregate **only Total rows** from the corresponding `.bed.gz` (haplotype-specific rows are excluded):

* `site_count` — number of Total rows emitted for this type
* `mod_count` / `unmod_count` / `inferred_unmod_count` / `diff_base_count` — sum of the same-named BED columns across all Total rows
* `observed_mod_distribution` — a 256-element histogram of the raw ML tag byte values (0–255) observed at emitted Total sites; `sum(observed_mod_distribution) == mod_count + unmod_count - inferred_unmod_count`, since inferred-unmod reads count toward `unmod_count` but do not have an ML entry
* `mod_score_distribution` — a 101-element histogram (bins 0–100) of the BED `mod_score` column rounded to the nearest integer percent at each emitted Total site; `sum(mod_score_distribution) == site_count`

Example (abbreviated; the distribution arrays are truncated with `…`):

```json
{
  "version": "1.0.0-<git-describe>",
  "datetime": "2026-04-21 18:00:00",
  "command": "methbat pileup --input-bam sample.bam --output-prefix /out/sample",
  "cli_settings": { "input_bams": ["sample.bam"], "output_prefix": "/out/sample", "…": "…" },
  "statistics": {
    "5mC":  { "site_count": 12345, "mod_count": 67890, "unmod_count": 54321,
              "inferred_unmod_count": 0, "diff_base_count": 12,
              "observed_mod_distribution": [0, 0, 1, "…", 5],
              "mod_score_distribution": [0, "…", 0] },
    "5hmC": { "site_count": 0, "mod_count": 0, "unmod_count": 0,
              "inferred_unmod_count": 0, "diff_base_count": 0,
              "observed_mod_distribution": [0, "…", 0],
              "mod_score_distribution": [0, "…", 0] },
    "6mA":  { "site_count": 0, "mod_count": 0, "unmod_count": 0,
              "inferred_unmod_count": 0, "diff_base_count": 0,
              "observed_mod_distribution": [0, "…", 0],
              "mod_score_distribution": [0, "…", 0] }
  }
}
```

# Downstream sub-commands
The unified pileup BED produced by this sub-command is the input for most downstream MethBat workflows:

* [methbat profile](./profile_guide.md) — per-region methylation labels (optionally with background Z-scores)
* [methbat segment](./segmentation_guide.md) — segment a single dataset into methylation categories
* [methbat joint-segment](./joint_segmentation_guide.md) — joint segmentation across a cohort
* [methbat signature](./signature_guide.md) — case-vs-control signature discovery
* [methbat report](./report_guide.md) — expected-vs-observed QC against pre-defined regions (e.g. imprinting)
* [methbat deconvolve](./deconvolution_guide.md) — cell-type deconvolution against a reference atlas

## Strand filter
Each downstream sub-command exposes a `--strand {combined|forward|reverse}` filter that selects which rows of this BED are considered (see also the `strand` column above):

* `combined` (default) - keep every row regardless of `strand`. This is the correct choice for symmetrical 5mC (all rows have `strand = .`) and is the pre-v1.0.0 behavior; for stranded modifications it folds `+` and `-` rows at the same coordinate into a single per-site entry.
* `forward` - keep only rows with `strand = +`.
* `reverse` - keep only rows with `strand = -`.

# IGV visualization of pileup
The BED files generated by `methbat pileup` are not suitable for direct IGV visualization.
To view the pileup scores in IGV, we recommend creating a [bedGraph file](https://genome.ucsc.edu/goldenpath/help/bedgraph.html) from the BED file.
The following command will extract the `Total` rows from a `methbat pileup` and reformat it for bedGraph:

```bash
# searches for "Total" rows and extracts coordinates and mod_score field
zgrep "Total" {prefix}.5mC.bed.gz |
  cut -f 1-3,7 |
  bgzip > {prefix}.5mC.bedgraph.gz
# index for faster IGV searches
tabix -p bed {prefix}.5mC.bedgraph.gz
```
