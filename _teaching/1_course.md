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

