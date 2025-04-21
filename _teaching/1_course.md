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
a practice dataset for trimming and mapping. If you are struggling with remembering CLI tools and commands,
visit the following [Github repo](https://github.com/RehanSaeed/Bash-Cheat-Sheet) that provides some nicely formatted cheat sheets for reference.


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


## Week 3: Quantification and Differential Expression

This week we will finally move away from BASH/UNIX and using remote computational servers, and pulling all of our data locally,
on machine, to perform all downstream analyses. For GENE 658, we will be using [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) to perform differnetial gene expression
\(hereon referred to as DGE\). Before we can begin the analysis, we need to ensure we have the following software installed:

 - [R](https://cran.r-project.org/), a computer langueage nearly all software we will use moving forward is coded in.
 - [RStudio](https://posit.co/download/rstudio-desktop/), an R IDE \(read as Interactive Development Environment\).
 - [Bioconductor](https://www.bioconductor.org/), a package repository/manager specializing in bioinformatics software.

Follow the links in order to get everything installed. First, download and install the R programming language. Next,
install RStudio. Once you have confirmed both R and RStudio have been installed properly, go ahead and open RStudio.
We will go over how to use Bioconductor in class.

RStudio is an R IDE. IDEs come in a lot of flavors, with various perks, but they all have the same underlying theme;
a single application that can help you manage code, variables, functions, packages, debug, and the list goes on and on.
We will not cover all of the various features and plugins availble in RStudio given the short length of this course. We
will focus on one main feature of RStudio, and it is really an extension of the R programming language itself.

When you installed R on your machine, it installed a plethora of files in your home or root directory. Most of these files
are needed to simply allow R to function properly on your computer. Others are some new odds and ends that have been integrated
into the R langueage since its original release. However many packages such as DESeq2, are not readily available. These have
to be downloaded and installed from the internet. 

In fact everytime, you need another R package, new tool, or even update and
old one to a new version, you will R will run a check to ensure there are no conflicts between packages. This can be quite the
tedious process and in some cases can be downright impossible depending on your hardware, and administrative level on your
system.

To avoid these issues, we are going to take advantage of something called the R Project manager. If you are at all familiar with
**virtual environments** from the Python language, the R project manager is somewhat similiar. Basically, the R Project Manager
\(`/renv`\), is a small piece of software that automatically manages depedencies, keeping files separate on a per project basis.
This brings a whole slew of benefits such as: portability, version control, project sharing, *etc*. We want to use `/renv` to
maintain our project files and environments so that we easily switch between projects. While we may not fully take advantage of
this during the course, you should certainly take advantage of this in your research projects.

Lastly, a couple of notes to go over about R and RStudio. In RStudio, a "project" can be thought of as simple as a single
analysis \(such as DGE\) of a dataset, or as complex as an entire workflow dedicates to analyzing your RNA-seq project from
exploratory data analysis, p-value calculations, pathway enrichment, visualization, and WGCNA. Suffice to say, this term
can mean widly different things depedning on who you ask. For this course, we will treat a "project" as the folder that
contains all files, data, and sub-directories related to performing a comprehensive analysis of your RNA-seq data. Another
term is "session". You can think of session as all of the data stored in RAM during an analysis. The count matrix for all
genes in your sample, the metadata for your samples, variance stabilized transformation of your gene counts, *etc*. All of
these items are stored in your machine's RAM and are viewable in the upper right box of the RStudio IDE. In most cases
you won't perform an entire project wide analysis in a single day. You may iterate over code, add new visualization,
or use new packages. In any of these cases, it would be nice to pick up the analysis right from where you left it the day
before. In order to do this, much like we would save a Word document we are drafting, we need to save the current state of
all the data we have generated in RStudio. We do this by saving the "session" to a file ending in .RData. That way the next
time we want to continue our analysis, or address a reviewer concern for an analysis we have in a manuscript in revision we
can pick up right were we left off. We will go over examples of this in class, but make sure you utilize projects and sessions
in RStudio.

For most of today we are going to walk through the [DESeq2 Vignette](https://master.bioconductor.org/packages/release/workflows/vignettes/rnaseqGene/inst/doc/rnaseqGene.html). You will notice there are two main tutorials
for RNA-seq analysis with DESeq2. The one I have linked above, is the older of the two, however I teach from it as I
believe it does a better job defining terms and showing clear workflows of analysis without assuming to much prior
knowledge.

In the class and for the homework we will now use the count matrixes of RNA isolated from rat derived, arterial-lining smooth
muscle cells. They can be downloaded [here](https://drive.google.com/drive/folders/1EoXmbJN3tFoF0GeS21ns0gCbhHSJDWkO?usp=sharing). This is a simple one **variable** test dataset, where the only difference between
the samples in this case is age; either "old" or "young". We will ignore the quantification of these variables for the
simplicity of class, however in practice this would likely be another variable to consider for mathematical modelling.

Please follow along in class and ask any questions you may have.

### Homework 3

Your homework for week 3 will be to uplaod a single .R or .Rmarkdown file with your R code to analyze this RNA-seq data.
Your R code should generate the following files:

 - A volcano plot of Fold change, and p-values for each gene in the count matrix.
 - A heatmap showing the counts for differentially expressed genes in all samples.
 - A PCA plot of visuzlaing vairance captured in PC1 and PC2 of all samples.

Please upload your homework to the [Canvas site](https://canvas.tamu.edu/courses/352642/modules/items/12483347).



