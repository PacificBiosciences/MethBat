<h1 align="center"><img width="300px" src="docs/images/logo_methbat.svg"/></h1>

<h1 align="center">MethBat</h1>

<p align="center">A battery of methylation tools for PacBio HiFi reads</p>

***

MethBat will aggregate and analyze CpG methylation calls made from [PacBio HiFi](https://www.pacb.com/technology/hifi-sequencing/) datasets.
Key features include:

* Creation of a methylation profile for a collection of genomic regions (e.g., CpG islands)
* Creation of cohort / background profiles from a collection of methylation profiles
* Analysis of cohort profiles comparing sub-groups (e.g., case v. control, male v. female)
* Segmentation of the CpGs in a dataset into Methylated, Unmethylated, or ASM regions
* Cell type deconvolution from bulk methylation data using reference atlases

Authors: [Matt Holt](https://github.com/holtjma), [Chris Saunders](https://github.com/ctsa)

## Early release warning
Please note that MethBat is still in early development. 
We are still tweaking the input / output file formats and making changes that can affect the behavior of the program.

## Availability
* [Latest release with binary](https://github.com/PacificBiosciences/MethBat/releases/latest)

## Documentation
* [Installation instructions](docs/install.md)
* [User guide](docs/user_guide.md)
* [Methods](docs/methods.md)
* [Data files](data/)

## Citation
MethBat does not currently have a pre-print or publication.

## Need help?
If you notice any missing features, bugs, or need assistance with analyzing the output of MethBat, 
please don't hesitate to open a GitHub issue.

## Support information
MethBat is a pre-release software intended for research use only and not for use in diagnostic procedures. 
While efforts have been made to ensure that MethBat lives up to the quality that PacBio strives for, we make no warranty regarding this software.

As MethBat is not covered by any service level agreement or the like, please do not contact a PacBio Field Applications Scientists or PacBio Customer Service for assistance with any MethBat release. 
Please report all issues through GitHub instead. 
We make no warranty that any such issue will be addressed, to any extent or within any time frame.

### DISCLAIMER
THIS WEBSITE AND CONTENT AND ALL SITE-RELATED SERVICES, INCLUDING ANY DATA, ARE PROVIDED "AS IS," WITH ALL FAULTS, WITH NO REPRESENTATIONS OR WARRANTIES OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, ANY WARRANTIES OF MERCHANTABILITY, SATISFACTORY QUALITY, NON-INFRINGEMENT OR FITNESS FOR A PARTICULAR PURPOSE. YOU ASSUME TOTAL RESPONSIBILITY AND RISK FOR YOUR USE OF THIS SITE, ALL SITE-RELATED SERVICES, AND ANY THIRD PARTY WEBSITES OR APPLICATIONS. NO ORAL OR WRITTEN INFORMATION OR ADVICE SHALL CREATE A WARRANTY OF ANY KIND. ANY REFERENCES TO SPECIFIC PRODUCTS OR SERVICES ON THE WEBSITES DO NOT CONSTITUTE OR IMPLY A RECOMMENDATION OR ENDORSEMENT BY PACIFIC BIOSCIENCES.
