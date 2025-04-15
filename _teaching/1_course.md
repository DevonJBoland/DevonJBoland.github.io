---
layout: page
title: GENE 658
description: Differential Expression Analysis
img: assets/img/teaching/volcanoplot.png
importance: 1
category: current
---

## Week 1: Read Trimming & Mapping

<div class="row">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/teaching/trimming_fig.jpg" title="read trimming" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A cartoon schematic that neatly describes why we need to trim raw sequencing reads.
</div>

One of the first steps of any RNA-seq data analysis, is to inspect the quality of your sequencing data.
To begin, we will need to become familiar with a couple of concepts:

  - FASTA/FASTQ format
  - Base call quality
  - Read trimming
  - QC metrics

In order to do this, we will take advantage of several open-source programs. Please
follow these links to the GitHub pages for various tools we will utilize in this course:

  - [Trim Galore](https://github.com/FelixKrueger/TrimGalore)
  - [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
  - [MultiQC](https://github.com/MultiQC/MultiQC)

One of the largest metrics we look at is the quality of base calling in our sequencing data.
Usually this will be a high number, especially in our current era of NGS/TGS seuqencing platforms.

However, never assume the data you are about to utilize is of good quality. Regardless of if you sequenced
the data, a lab member did, or a collaborator is sharing the data with you; you must always
inspect the quality of the data. Base quality scoring is calcuated as a Phred score.

[Learn more about Phred scores and how they are calculated here](https://en.wikipedia.org/wiki/Phred_quality_score)

Please download the following [data](https://drive.google.com/file/d/16fJ42CBaj0Ss-pEmpBVpV2ub0DsTP-_J/view?usp=sharing) which we will use as
a practice dataset for trimming and mapping.


### Homework #1
For homework #1 you have just started your new job at Boland Bioinformatics \(The Premier Bioinformatics Company!\). You are being trained
on the workflows and pipelines you will be expected to deploy and maintain. Your first day has you learning how to perform DataQC on all
incoming datasets. However the [Snakemake](https://snakemake.readthedocs.io/en/stable/) pipeline the company normally employs has broken,
as Phil issued a `rm -r *` command in the pipeline directory by accident! You being the quick thinker you are, decide to write your own script
to continue data processing without interuppting services to clients. You know the following about the software you must use, and the datasets:

  - The company uses TrimGalore and MultiQC for trimming and reporting
  - The data can consist of a mixture of paired-end and single-end read sets

Your job for the homework is to write a single job script \(.sh\) file that you would execute on the FASTER hprc system at TAMU. This single
job script needs to be able to parse all your files in a single directory, perform trimming, and create a final HTML report for easy review.
You will find the read data sets you need to analyze [here](https://drive.google.com/file/d/17DAsMYF7L9lKrYbcV8HhRIs0BmQksYZ_/view?usp=sharing) You will upload this assignment to Canvas under the module titled Week #1. 
Email me [devonjboland@tamu.edu](mailto:devonjboland@tamu.edu) if you have any questions.

####
##

## Week 2: Read Mapping/Alignment

<div class="row">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/teaching/read_mapping_example.png" title="read mapping" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Mapping reads to a reference or de novo assembly. Reprinted from Macmillan Publishers Ltd: Nature Biotechnology. Haas BJ and Zody MC. Advancing RNA-seq analysis. 28:421-423, copyright 2010 (10).
</div>

After we have finished trimming and inspecting the quality of our read sets, we are now ready to map our reads to a reference.
Typically this involves either mapping our reads back to genome or transcriptome assembly. Both approaches typically trade
pros and cons and have certain advantages over the other. We will cover two mappers/aligners in class:

  * [STAR](https://github.com/alexdobin/STAR?tab=readme-ov-file)
  * [Salmon](https://combine-lab.github.io/salmon/about/)

While STAR is now able to directly align RNA-seq reads to both a genome or transcriptome reference, in its original release
it was only able to map to genome reference. Conversly, Salmon is only designed to align reads to a transcriptome reference.
There are a lot of differences between the tools, and they will not be discussed here, if you are interested in learning
about these differences, I encourage you to read the respecitve papers for each tool. For the sake of this class, we will
use STAR to align RNA-seq reads to a genome reference, and Salmon to align RNA-seq reads to a transcriptome reference.

To use STAR, we need to obtain reference files, in this case a genome assembly and gene transfer format \(GTF\) files.
We will obtain these files from the [Ensembl](https://useast.ensembl.org/index.html) database. I prefer using Ensembl formatted files, as most tools
that we cover for downstream analysis will support Ensembl acession IDs out of the box. After we download these files,
we must index our references. This is a common step in nearly all mappers and aligners, and creates a data format structure
that allows aligners to quickly perform lookups in large assembly files on the fly instead of iteratively searching across 
the entire genome starting at the begining of chromsome 1 for each read. Essentially, this results in hardly any decrease
in alignment accuracy and vastly increases the speed of the alignment. 

After we index the reference, we are free to begin mapping our trimmed reads below is the example script we wrote together
in class:

```bash
#!/bin/bash
#SBATCH --job-name=gene658_trim_test
#SBATCH --time=1:00:00
#SBATCH --nodes=1          # max 32 nodes for partition cpu
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=50G
#SBATCH --output=stdout.%x.%j.txt
#SBATCH --error=stderr.%x.%j.txt

### Declare Variables ###
threads=$SLURM_CPUS_PER_TASK

### Begin Trimming ###

# Load modules
ml GCCcore/11.3.0 Trim_Galore/0.6.10

# run trim galore
cat files.txt | while read line;
do
	if [[ -f "${line}_R2.fastq.gz" ]]; then
		trim_galore --paired -j ${threads} --fastqc ${line}_R1.fastq.gz ${line}_R2.fastq.gz
	else;
		trim_galore -j ${threads} --fastqc ${line}_R1.fastq.gz
done
ml purge

# Combine reports with multiqc
ml GCC/13.2.0  OpenMPI/4.1.6 MultiQC/1.27.1
multiqc .
ml purge
```
### Homework #2

For homework #2 your continuing your new job at Boland Bioinformatics \(The Premier Bioinformatics Company!\). Phil
in his continuing incompetency deleted the [Snakemake](https://snakemake.readthedocs.io/en/stable/) pipeline the company normally employs for read alignment as well.
Continuing on your quick decisioin making roll, you decide to write your own script to perform the mapping of same reads
from Homework #1. You know the following information:

  - All of the reads were sequencing from rat vascular smooth muscle cells \(ignore the file naming of the reads\)
  - You have both single-end and paired-end reads.

Your job for the homework is to write a single job script \(.sh\) file that you would execute on the FASTER hprc system at TAMU.
This single job script needs to be able to parse all your files in a single directory and using STAR
create a reference index, and map the trimmed reads to said reference. Lastly, you want to create a nice mapping 
alignment graph using MultiQC as we did in class. You will upload this assignment to Canvas under the module titled Week #1
Email me [devonjboland@tamu.edu](mailto:devonjboland@tamu.edu) if you have any questions.