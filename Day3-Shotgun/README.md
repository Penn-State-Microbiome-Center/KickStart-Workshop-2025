# Day 3: Metabolomics and Metagenomics

## Metabolomics:

Intro to Metabolomics (Josh Kellogg) - [pdf](MicrobiomeKickstartLecture_Kellogg.pdf)

## Whole genome shotgun metagenomics analysis

### Reminder if you forget how to access the ACI ROAR Collab cluster

#### Graphical user interface
1. Nagivate to [rcportal.hpc.psu.edu](https://rcportal.hpc.psu.edu/)
2. In the "Interactive Apps", choose "Roar Collab RHEL8 Interactive Desktop"
5. Under "Number of hours", select 10
6. Change the number of nodes to 1, the number of cores to 4, and the memory to 64
7. Click "Launch"

After that, open a terminal (little black icon on the top bar, next to the firefox icon), and then enter the following commands:
```
module load anaconda
conda activate /storage/icds/RISE/training/microbiome/2024/envs/microbiome
```
### Shotgun metagenomic analysis
This portion of the workshop will cover a few of the basic computational approaches to studying WGS metagenomic data. 
In contrast to 16S rRNA studies (which have platforms like QIIME), there is no agreed-upon "all-in-one" analysis platform for WGS metagenomic analysis.
There are a few pretty comprehensive platforms such as:
1. [MEGAN](https://uni-tuebingen.de/fakultaeten/mathematisch-naturwissenschaftliche-fakultaet/fachbereiche/informatik/lehrstuehle/algorithms-in-bioinformatics/software/megan6/) 
a 16S and WGS platform, designed for GUI use (i.e. hard to work with on a cluster/server)
2. [Anvi'o](https://merenlab.org/software/anvio/) a WGS platform, designed for use on a server. A robust, capable platform, but with a somewhat steep learning curve and a number 
of the analyses are either non-standard or not state-of-the-art.

As such, we will be covering some of the current state-of-the-art (according to the recent [CAMI2 manuscript](https://www.nature.com/articles/s41592-022-01431-4)) stand-alone tools. We will be covering the following techniques

![Main slides](https://user-images.githubusercontent.com/6362936/128754520-4e2852aa-52b4-43a5-9e68-4e6f4f030379.png)


### Taxonomic profiling
In the [Taxonomic Profiling](TaxonomicProfiling.md) section, we will be covering a couple of tools capable of answering the question:
"Which taxa are present in my metagenome, and at what relative abundance?

### Assembly
In the [Assembly](Assembly.md) section, we will cover a couple tools that will take as input your short sequences and assemble them into contigs and/or scaffolds. 

#### Binning
In the [Binning](Binning.md) section, we will cover tools that either:
1. Cluster the assembled contigs into putative single genome origin bins (genome binning)
2. Cluster the assembled contigs into bins which correspond to different taxa (taxonomic binning)

## Please proceed now to the [Taxonomic Profiling](TaxonomicProfiling.md) section

