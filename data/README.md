# Data
This folder contains data resources for use with MethBat.

## CpG profiles
These files contain background / cohort CpG profiles that can be provided to MethBat to describe coordinates of regions of interest.
Note that methylation profiles can be tissue specific depending on the region. 
For example, running a blood sample dataset against the our HPRC-based (cell-line based) background profile is likely to lead to more false positive / negative results with respect to comparison classifications.
These cohort profiles are subject to change with new releases as we refine the datasets, upstream processing pipeline, and MethBat.
Where possible, we recommend building custom cohort profiles from datasets with identical sample type and processing.

* [cpgIslandExt.sorted.hg38.tsv](./cpgIslandExt.sorted.hg38.tsv) - TSV-reformatted version of the CpG island track from UCSC for hg38 that is essentially a "blank" profile containing only the coordinates of the islands. If building a new background / cohort profile, this is the data file to provide when building the initial single-dataset profiles.
* [meth_profile_model.tsv](./meth_profile_model.tsv) - A pre-built background profile for the above CpG island set made from 75 HiFi datasets. 65 of these datasets are from HPRC cell lines, and the remaining 10 are from MGISTL pangenome blood samples. 45 of the datasets are annotated as "FEMALE", and the remaining 30 are "MALE". The profile includes "ALL", "FEMALE", and "MALE" data categories. All datasets were aligned using pbmm2, variant called using DeepVariant, and haplotagged with WhatsHap. This profile was derived from outputs from the "model" mode of pb-CpG-tools. This is the recommended pre-built profile.
