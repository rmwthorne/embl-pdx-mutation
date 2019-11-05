# Workshop: *In silico* genomic mutation analysis

## Summary

This workshop is intended to introduce the command-line tools 
and file types used in a simple bioinformatics workflow 
for mutation detection with Patient Derived Xenograft (PDX) 
Whole Exome Sequencing (WES) data. It was written for the workshop for Humanized Mice in Biomedicine (2019) in EMBL, Heidelberg. 

## Prerequisites

- Basic scientific background of PDX models and mutations
- You should have [Docker](https://www.docker.com/get-started) and [ORCA](https://github.com/bcgsc/orca) installed (done for you)
  - If you want to set up on another computer, you can install Docker Community Edition (CE) following the instructions in the link above
  - You can install ORCA by running `docker pull bcgsc/orca`

## Overview

- [Part 1](./test.md): This will introduce you to the command-line
- Part 2: You will use [ORCA](https://doi.org/10.1093/bioinformatics/btz278), a [Docker](https://www.docker.com/get-started)-based bioinformatics environment to align some `FASTQ` files from the [NCI Patient-Derived Models Repository](https://pdmr.cancer.gov/models/database.htm) (PDMR)
- Part 3: You will use `freebayes` to call genomic variants

Time plan:
- 14:30 Part 1
- 15:00 Part 2
- 15:30 Part 3

## Learning Objectives

1. Become familiar with the Command-Line Interface
1. Understand the common file formats used in genomic variant detection
1. Perform sequencing read alignment
1. Run a variant caller to identify Single Nucleotide Variations (SNVs) and insertion/deletion mutations (indels)

For a lot of scientists, bioinformatics is a mysterious and daunting black box. My aim for this workshop is not *necessarily* to get you performing variant analysis in an hour (though there's content here ready if you wan to!), but rather to lift up the curtain and give you an idea of what's actually happening in that black box.

There is a lot to cover even with a simplified variant calling pipeline, 
and it is unlikely we will cover everything in the 1.5 hours we have, however, the tutorial should give you the starting tools to takle the rest of it. You are encouraged to finish the workshop in your own time should you wish. I will respond to any issues or questions you have, on Humanised Mice slack, the github repository, or by email if you prefer.

## Updates

The workshop will continue to be updated, so please feel free to submit questions or suggestions. Let's make a useful resource for other PDX scientists interested in analysing their own data!

You can update your version of this tutorial any time you like, by running the following command when you're in the tutorial directory. If you don't know how to run commands yet, you will do soon!

```sh
git pull
```


# Part 1: Welcome to the Command-Line

When working with bioinformatics, it is impossible to avoid working with
the Command Line Interface (CLI). 
You will use it to access all of the tools of the trade. 
At first it will be daunting, and will take a while to get things done, 
but soon enough you will get more comfortable with it.


## 🎯 Task 1

First, complete the exercises in Amblina's [Command-line Tutorials](https://katacoda.com/amblina):

1. Introduction to the Command-Line
1. Beginner's guide to pagers
1. Familiarising yourself with the unfamiliar
1. Reading files again


# Part 2: Sequencing Read Alignment

We will now download some files to analyse. These are from the sample **OT-FWCP10** from the PDMR database, 

From this [location](https://pdmdb.cancer.gov/pls/apex/f?p=101:24:0::NO:::), accessed on 2019/10/21:

| Patient ID | Specimen ID | Sample ID | CTEP SDC Code | CTEP SDC Description | Disease Body Location | PDM Type | VCF | VCF Version | Read1 FASTQ | Read2 FASTQ | FASTQ Version |
| - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| 994819 | 140-R | OT-FWCP10 | 10006190 | Invasive breast carcinoma | Breast | PDX | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.vcf) (25mb) | 2.0.1.51.0 | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.R1.FASTQ.gz) (5gb) | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.R2.FASTQ.gz) (5gb) | 2.0.1.51.0 |

1. In your current working directory, create a directory called `fastqs`.
1. Change directory into `fastqs`.
1. Download the fastq files using the following commands:
    - `curl www.google.com -O`
    - `curl www.google.com -O`
1. Can you find out what the `-O` (capital 'o', not zero!) flag is telling `curl` to do?


We will introduce 3 frequently used bioinformatic file formats in this tutorial:

| Filetype | Contains | Description |
| - | - | - |
| FASTA| Sequences | A collection of DNA sequences. We store reference genomes in this format. |
| FASTQ | Reads | A collection of signals ("reads") that represent DNA sequences, generated from a high-throughput sequencer. Contains quality scores for each read. |
| SAM | Alignments | A collection of reads that have been aligned to a reference genome. | 
 | VCF | Mutations | A list of variations (i.e. mutations) found in an alignment. |

From the descriptions, you might guess that these build on each other. We will start with FASTA and FASTQ files, and generate SAM alignments from them. From the SAM files, we will generate a VCF that will contain processed information about the mutations we have found in our sample.

## Containerised Analysis using ORCA

To start working in ORCA, use the following command:

```sh
docker run --interactive --tty --volume $HOME:$HOME --workdir $HOME bcgsc/orca
```

You will spin up your virtual linux operating system and see your prompt within ORCA's shell:

```sh
root@b4601473c56e:/Users/rthorne#
```

You can use `exit` to close the container and return to your system shell at any time.

## FASTQ File Format

We now have some FASTQ files. These might be the first files containing genomic data that you've used. They look just like other plain text `.txt` files.

We are going to use ORCA for these exercises. You can think of it as a tiny, self-contained little linux computer working inside your current machine. There are benefits to using such an environment, such as for ease of program installation and reproducibility.

1. First let's unzip and call `head` to see the first 10 lines of `read_1.fastq.gz`: 
    - `$ gunzip read_1.fastq.gz`
1. Each read is represented by 4 lines:

```
e.g.
identifier:    @61CC3AAXX100125:7:72:14903:20386/1
read sequence: TTCCTCCTGAGGCCCCACCCACTATACATCATCCCTTCATGGTGAGGGAGACTTCAGCCCTCAATGCCACCTTCAT
separator:     +
quality score: ?ACDDEFFHBCHHHHHFHGGCHHDFDIFFIFFIIIIHIGFIIFIEEIIEFEIIHIGFIIIIIGHCIIIFIID?@<6
```

You can see that is is a series of repeating lines:

1. Identifier line (`@D007`...)
1. Read sequence line (`NTGAA`...) Now this makes sense to a biologist. This is the nucleotide base that has been decided to be the most likely ('called') by the base caller. If it can't decide, you'll notice, it assigns an `N`. Obviously, these are arranged left → right, 5' → 3'.
1. Separator line (`+`) This is the strand that the read came from. Either the `+` or `-` strand.
1. Quality score line . This tells you about the confidence in data quality for each base on line 2 below. Each character maps to a numeric quality score value.

## SAM File Format

We are going to align the reads in the fastq files to a reference genome using a CLI tool called `bowtie`.

<!-- [Burrows-Wheeler Algorithm](http://bio-bwa.sourceforge.net/), using a CLI tool called `bwa`. -->

```sh
(Our usage:)
bowtie --sam <reference> -1 <R1.fastq> -2 <R2.fastq> <output.sam>

bowtie --sam --chunkmbs 200 reference/igenomes/Homo_sapiens_NCBI_GRCh38/Homo_sapiens/NCBI/GRCh38/Sequence/BowtieIndex/genome -1 fastq/*R1* -2 fastq/*R2* aligned.sam
```

# Part 3: Mutation Detection

## VCF File Format

Now that we have aligned our reads to the reference genome, we can perform **variant calling** - we can identify where there are mismatches between the two, which suggest a mutation in the experimental sample. We will use the `gatk` toolkit to do this and generate a VCF file.

You can download a vcf of the above sample. If we call `head file.vcf` then we can preview the contents of the file. VCF files are a tab-delimited file. You can think of them like Excel spreadsheets, except they use tabs to separate columns and returns to separate rows. You can even open it in Excel or another spreadsheet program.

# Going Further

The [vcfR](https://knausb.github.io/vcfR_documentation/visualization_1.html) package for [R](https://en.wikipedia.org/wiki/R_(programming_language))

Docker is a great tool that can be used to tackle one of the long-standing problems of bioinformatics analyses: Reproducibility. For an example of why, you can see the [recent story about python in the news](https://arstechnica.com/information-technology/2019/10/chemists-discover-cross-platform-python-scripts-not-so-cross-platform/)[Neupane et al. 2019](https://pubs.acs.org/doi/10.1021/acs.orglett.9b03216)


# Acknowledgements

Build upon, and acknowledge the work of others, and don't repeat yourself when you don't have to.

- Thank you to Amblina for the use of her excellent [Command-line Tutorials](https://katacoda.com/amblina)
- Thanks to the authors of [this tutorial](https://melbournebioinformatics.github.io/MelBioInf_docs/tutorials/variant_calling_galaxy_1/) by [Melbourne Bioinformatics](http://www.melbournebioinformatics.org.au/). I have based this workshop around their excellent content. I encourage you to check it out if you would like an approach using the GUI bioinformatics tool, [Galaxy](https://usegalaxy.org)

# Glossary
<dt>Docker</dt><dd>A lightweight virtualisation tool. You can think of it as a miniature simulated linux computer running within your computer)