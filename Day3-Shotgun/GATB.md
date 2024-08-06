# GATB
- [GATB](#gatb)
  - [Installation and setup](#installation-and-setup)
  - [Running GATB](#running-gatb)
  - [Assess the assembly quality](#assess-the-assembly-quality)

GATB, aka The Genome Analysis Toolbox with de-Bruijn Graph is a quite flexible toolkit of various genomic analysis tools. check out their [software](https://gatb.inria.fr/software/) 
page if you would like to count k-mers in, error correct, assemble, compare, etc. genomic and metagenomic samples. One of the main authors of this toolkit is Rayan Chikhi, a 
former postdoc of Paul Medvedev's (CSE & BMB)!

While there are a lot of different goodies we could look at, we will focus just on their pipeline for _de novo_ metagenome assembly which combines the tools of 
[Bloocoo](https://gatb.inria.fr/software/bloocoo/) error correction, [Minia](https://github.com/ksahlin/BESST) assembly, and [BESST](https://gatb.inria.fr/software/minia/) scaffolding. Similar to MEGAHIT, the core assembler Minia is a multi-k-mer size assembler.

## Installation and setup
Let's use the usual folder structure
```
cd ~
mkdir GATB_analysis
cd GATB_analysis
mkdir scripts output data
```
This pipeline is installed in a different conda environment, so please do the following to activate it on your system
```
module use /gpfs/group/RISE/sw7/modules
module load anaconda
conda activate gatb
```

If you want to install it in a different location, here are the steps:
```
conda create -y -n gatb scipy numpy mathstats pysam
git clone --recursive https://github.com/GATB/gatb-minia-pipeline
cd gatb-minia-pipeline ; make test
export PATH="$PWD:$PATH"
```

Lastly, we will again obtain the (same) data:
```
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2022/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```

## Running GATB

To run the GATB pipeline with default settings, we can do the following:
```
cd output
gatb -s ../data/SRS014464-Anterior_nares.fasta -o default
cd ..
```
Note that GATB wants an output _prefix_ as opposed to an output folder. This is why we need to be in the output directory when we call GATB.
If we wanted to, we could run GATB on all the data, but for the sake of time, let's stick with just this one sample.

As with the other tools, the final contigs (or scaffolds, in this case) are contained in a single file called `output.fasta`. We will next use QUAST to get some stats 
about the quality of this assembly.

## Assess the assembly quality
Let's activate our QUAST environment:
```
conda deactivate
conda activate microbiome2
```

And then run QUAST on the assembly:
```
quast -o output/quast_out -m 250 --circos --glimmer --rna-finding --single data/SRS014464-Anterior_nares.fasta output/default.fasta
```
And view the output:
```
firefox output/quast_out/report.html
```
![GATB](https://user-images.githubusercontent.com/6362936/128284961-4bd6722d-08d1-4f47-989d-342a22509754.PNG)


**How does this assembly compare to the previous ones?**

**Try BLASTing the longest contig. Any guesses where this contig might be from?**

# Please proceed to the [Binning](Binning.md) section
