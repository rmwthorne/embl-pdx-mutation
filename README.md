# Workshop: *In silico* genomic mutation analysis

## Summary

This workshop is intended to introduce the command-line tools 
and file types used in a simple bioinformatics workflow 
for mutation detection with Patient Derived Xenograft (PDX) 
Whole Exome Sequencing (WES) data. It was written for the workshop for Humanized Mice in Biomedicine (2019) in EMBL, Heidelberg. 

## Prerequisites

- Basic scientific background of PDX models and mutations
- You should have [Docker](https://www.docker.com/get-started) and [ORCA](https://github.com/bcgsc/orca) installed (done for you)

## Overview

- [Part 1](#part-1-welcome-to-the-command-line): This will introduce you to the command-line
- [Part 2](#part-2-sequencing-read-alignment): You will use [ORCA](https://doi.org/10.1093/bioinformatics/btz278), a [Docker](https://www.docker.com/get-started)-based bioinformatics environment to align some `FASTQ` files from the [NCI Patient-Derived Models Repository](https://pdmr.cancer.gov/models/database.htm) (PDMR)
- [Part 3](#part-3-mutation-detection): You will use `freebayes` to call genomic variants

Time plan:
- 14:30 Part 1
- 15:00 Part 2
- 15:30 Part 3

## Learning Objectives

1. Become familiar with the Command-Line Interface
1. Understand the common file formats used in genomic variant detection
1. Perform sequencing read alignment
1. Run a variant caller to identify Single Nucleotide Variations (SNVs) and insertion/deletion mutations (indels)


For a lot of scientists, bioinformatics is a mysterious and daunting black box. My aim for this workshop is not *necessarily* to get you performing variant analysis in an hour (though there's content here ready if you want to!), but rather to lift up the curtain and give you an idea of what's actually happening in that black box, and convince you that you can do it too.

There is a lot to cover even with a simplified variant calling pipeline, 
and it is unlikely we will cover everything in the 1.5 hours we have, however, the tutorial should give you the starting tools to takle the rest of it. You are encouraged to finish the workshop in your own time should you wish. I will respond to any issues or questions you have, on Humanised Mice slack, the github repository, or by email if you prefer.

## Advanced Objectives

Already comfortable with the command line? Here's what you can expect to add to your bioinformatics repetoire:

- A troubleshooting strategy for most situations
- Modern best practices regarding reproducible analyses - using and creating your own docker images
- An introduction to using R for downstream analysis and presentation

## Updates

The workshop will continue to be updated, so please feel free to submit questions or suggestions. Let's make a useful resource for other PDX scientists interested in analysing their own data!

## Setting up yourself

- If you want to set up on another computer, you can install Docker Community Edition (CE) following the instructions in the link above
- You can then install ORCA by running `docker pull bcgsc/orca` anywhere
- Without access to a computing cluster, Galaxy remains a great way to learn bioinformatics. We will include links to relevent tutorials to keep you going, now that you know what's happening 'under the hood'

If you have any questions or problems feel free to contact me.

# Part 1: Welcome to the Command-Line

When working with bioinformatics, it is impossible to avoid working with
the Command Line Interface (CLI). 
You will use it to access all of the tools of the trade. 
At first it will be daunting, and will take a while to get things done, 
but soon enough you will get more comfortable with it.


## 🎯 Task 1

First, complete the exercises in [Amber Wright's Command-line Tutorials](https://katacoda.com/amblina):

1. Introduction to the Command-Line
1. Beginner's guide to pagers
1. Familiarising yourself with the unfamiliar
1. Reading files again


# Part 2: Sequencing Read Alignment

Some files for analysis have already been downloaded to your computer in the `/data` directory. Using your newfound command-line skills, we are going to investigate them and show you what some of the files actually look like.

These FASTQ files are from the sample [OT-FWCP10](https://pdmdb.cancer.gov/pls/apex/f?p=101:24:0::NO:::) from the PDMR database. In the interests of time, I have isolated only those reads that should map to p13.1 of chromosome 17 (chr17:6,600,000-10,600,000, around 4Mb worth). The reference genome (build hg38) is from Illumina's [igenomes](https://support.illumina.com/sequencing/sequencing_software/igenome.html).

## 🎯 Task 2

This will build on what you learned in Part 1.

1. Change directory into `/data`
1. List the files in the directory
1. Let's organise things a bit: 
     1. Make a `fastqs` directory
     1. Move the FASTQ files into it. You can use `mv` to do this:
        ```sh
        mv r1.fastq fastqs/
        mv r2.fastq fastqs/
        ```
1. List the files that are in `fastqs/`

<!-- 
| Patient ID | Specimen ID | Sample ID | CTEP SDC Code | CTEP SDC Description | Disease Body Location | PDM Type | VCF | VCF Version | Read1 FASTQ | Read2 FASTQ | FASTQ Version |
| - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| 994819 | 140-R | OT-FWCP10 | 10006190 | Invasive breast carcinoma | Breast | PDX | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.vcf) (25mb) | 2.0.1.51.0 | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.R1.FASTQ.gz) (5gb) | [Open](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.R2.FASTQ.gz) (5gb) | 2.0.1.51.0 | 
-->

## Containerised Analysis using ORCA

We are going to use ORCA for these exercises. You can think of it as a tiny, self-contained little linux computer working inside your current machine. There are benefits to using such an environment, such as for ease of program installation and reproducibility.

To start working in ORCA, use the following command:

```sh
docker run --interactive --tty --volume /data:/Users/training --workdir /Users/training bcgsc/orca
```
There are a number of options here that we've called docker with:

- `--interactive` allows you to interact with the bash interface of the container
- `--tty` opens a tty terminal session (Ross needs to update this)
- `--volume <OS directory>:<docker directory>` mounts a *volume* to docker. This just allows you to access the fastq files from your machine from within the docker environment
- `--workdir` sets the current working directory for when you start the container

You will spin up your virtual linux operating system and see your prompt within ORCA's shell:

**All paths from now on will be relative paths from your `/data` directory**

```sh
root@b4601473c56e:/Users/training#
```

You can use `exit` to close the container and return to your system shell at any time.

<!-- 
## FASTQ File Format

We now have some FASTQ files. These might be the first files containing genomic data that you've used. They look just like other plain text `.txt` files.

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
1. Quality score line . This tells you about the confidence in data quality for each base on line 2 below. Each character maps to a numeric quality score value. -->

## SAM File Format

SAM files describe where your sequencing reads map to on a reference genome. Here is an example of a line that represents an alignment. Most of these fields do not convey much information, but the following fields make intuitive sense:

- 3. (`chr17`) 'RNAME': this shows the reference chromosome that was mapped to.

- 4. (`10450470`) ''POS': the coordinate of the leftmost read base

```
D00748:170:CD31MANXX:5:1101:1136:89194	99	chr17	10450470	255	124M	=	10450535	189	CTCTGTGCGCTNGATGGCGTCCGTCTCGTACTTGGTCCNCCACTGGGCAACCTCACTGTTNNCCTTGGACATTCCCCTCTGCAGCTCAGCCTTGGCTTCCTGCTCCTCCTCATACTGTTCCCGC	BBBBBFFFFFF#<BFBFFFFFFFFBFFFFFFFFFFFBB#<<FBFF/FBFF/BFFFFFFFF##<<<FF/<F/FFFBFFFFFFFFFFFFFFFFFFFFFFFFFFBFBBBFFFFFFFBBFFBFBBBBB	XA:i:1	MD:Z:11G26T21G0G62	NM:i:4	XM:i:2
```

## 🎯 Task 3

1. Align the reads in the fastq files to a reference genome using a CLI tool called `bowtie`.

    We will use the command like so:
    `bowtie --sam <reference> -1 <R1.fastq> -2 <R2.fastq> <output.sam>`

    ```sh
    bowtie --sam --chunkmbs 200 Homo_sapiens/UCSC/hg38/Sequence/BowtieIndex/genome -1 fastqs/r1.fastq -2 fastqs/r2.fastq aligned.sam
    ```
    The options we pass to `bowtie` are:
    - `--sam` indicates we want the output to be in SAM format
    - `--chunkmbs 200` is required to prevent our machines running out of memory while trying to find the alignment for each read
    - `Homo_sapiens/UCSC/hg38/Sequence/BowtieIndex/` is the first unnamed option we use. It points to our reference genome, as described above
    - `-1` points to the location of our first FASTQ file - this contains the first mates of each paired-end read
    - `-2` likewise points to the location of our second FASTQ file
    - `aligned.sam` is our second unnamed option. It is the name that we want the output file to be
2. We need to do a couple of operations on our SAM file before it is ready for mutation detection, we need to **compress** and **sort** the data.
3. Compress the file. We are going to use the binary SAM (bam) format:
    ```sh
    samtools view -b aligned.sam > aligned.bam
    ```
4. Sort the file. We can do this by using samtools again:
    ```sh
    samtools sort aligned.bam -o aligned.bai
    ```

# Part 3: Mutation Detection

## VCF File Format

Now that we have aligned our reads to the reference genome, we can perform **variant calling** - we can identify where there are mismatches between the two, which suggest a mutation in the experimental sample.

## 🎯 Task 4

1. Call variants using `freebayes`:
```sh
freebayes -f Homo_sapiens/UCSC/hg38/Sequence/BowtieIndex/genome.fa aligned.bai > test_mutations.vcf
```
3. If we call `head test_mutations.vcf` then we can preview the contents of the file. VCF files are a tab-delimited file. You can think of them like Excel spreadsheets, except they use tabs to separate columns and returns to separate rows. You can even open it in Excel or another spreadsheet program.
2. Take a vcf that has been generated from the full read dataset [here](ftp://dctdftp.nci.nih.gov/pub/pdm/994819~140-R~OT-FWCP10~v2.0.1.51.0~WES.vcf) and upload it to [CRAVAT.us](https://cravat.us)


# Going Further

The [vcfR](https://knausb.github.io/vcfR_documentation/visualization_1.html) package for [R](https://en.wikipedia.org/wiki/R_(programming_language)). To use it, we will use [RStudio](https://rstudio.com/) (via [Rocker](https://www.rocker-project.org/)), which is a nice editor for working in R with.

```
docker run -e PASSWORD=<YOUR_PASSWORD> -p 8787:8787 rocker/rstudio:3.6.1
```

1. Navigate your web browser to [localhost:8787](http://localhost.com:8787)
1. Log into the rstudio interface with username / password `rstudio`/`<YOUR_PASSWORD>`
1. Install the package with `install.packages("vcfR")`⏎

## Troubleshooting

Whoops, everything looked like it was going fine, but it says that the install terminated somewhere! 🙊 We are now going to cover an essential programming skill: *troubleshooting*. Every programmer and bioinformatician - beginner or seasoned veteran - has to do this, and you'll have to learn sooner or later too.

Simply, we are going to use a search engine with two pieces of information, 1) The most relevent part of the debug error and 2) the context in which we're encountering it. In this case I searched for:
> vcfr zlib.h: No such file or directory  #include <zlib.h>
1. We find a [similar question](https://stackoverflow.com/questions/18148075/compilation-error-missing-zlib-h/38277221) asked by someone on stackoverflow, and a couple of answers. [This answer](https://stackoverflow.com/a/38277221) seems to be the key to solving it.
1. According to the discussion, it appears that we're missing a dependency, a `C` library called `zlib`. The answer details how we can install this dependency and resolve our installation, run:
```sh
# Note: Run in container
sudo apt-get update
sudo apt-get install libz-dev
```
3. Now that's installed, we can return to RStudio and run the `install.packages` command again to complete the install and load the library
```r
install.packages("vcfR")
library("vcfR")
``` 
4. Now we are ready to follow the vignette for `vcfR`

Docker is a great tool that can be used to tackle one of the long-standing problems of bioinformatics analyses: Reproducibility. For an example of why, you can see the [recent story about python in the news](https://arstechnica.com/information-technology/2019/10/chemists-discover-cross-platform-python-scripts-not-so-cross-platform/)[Neupane et al. 2019](https://pubs.acs.org/doi/10.1021/acs.orglett.9b03216)

# 🙂 Feedback

Any feedback would be very welcome! Three short questions, no required answers, completely anonymous!

[Feedback form](https://forms.gle/CjQHP2LSgBGePhX7A)

# Acknowledgements

Build upon, and acknowledge the work of others, and don't repeat yourself when you don't have to.

- A huge thank you to Amber Wright for the use of her excellent [Command-line Tutorials](https://katacoda.com/amblina)
- Thanks to the authors of [this tutorial](https://melbournebioinformatics.github.io/MelBioInf_docs/tutorials/variant_calling_galaxy_1/) by [Melbourne Bioinformatics](http://www.melbournebioinformatics.org.au/). I have based this workshop around their excellent content. I encourage you to check it out if you would like an approach using the GUI bioinformatics tool, [Galaxy](https://usegalaxy.org)

# Glossary
<dt>Docker</dt><dd>A lightweight virtualisation tool. You can think of it as a miniature simulated linux computer running within your computer)</dd>

# Filetypes Reference

We will introduce 3 frequently used bioinformatic file formats in this tutorial:

| Filetype | Contains | Description |
| - | - | - |
| FASTA| Sequences | A collection of DNA sequences. We store reference genomes in this format. |
| FASTQ | Reads | A collection of signals ("reads") that represent DNA sequences, generated from a high-throughput sequencer. Contains quality scores for each read. |
| SAM | Alignments | A collection of reads that have been aligned to a reference genome. | 
 | VCF | Mutations | A list of variations (i.e. mutations) found in an alignment. |

From the descriptions, you might guess that these build on each other. We will start with FASTA and FASTQ files, and generate SAM alignments from them. From the SAM files, we will generate a VCF that will contain processed information about the mutations we have found in our sample.

