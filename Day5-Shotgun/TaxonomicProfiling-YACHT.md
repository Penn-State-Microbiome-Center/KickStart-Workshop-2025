
# mOTUs2 Tutorial
===========================
- [mOTUs2 Tutorial](#motus2-tutorial)
- [Installation and setup](#installation-and-setup)
  - [File structure and data:](#file-structure-and-data)
- [Create taxonomic profiles](#create-taxonomic-profiles)
  - [Input files](#input-files)
  - [Run multiple samples](#run-multiple-samples)
- [Comparing differences between tools](#comparing-differences-between-tools)
  - [Supporting evidence](#supporting-evidence)

[mOTUs2](https://motu-tool.org/index.html) is a tool for profiling the taxonomic composition of microbial communities from metagenomic
shotgun sequencing data.

This tool has a good balance of sensitivity and specificity. It is based on marker genes, but in contrast to MetaPhlAn, it uses single nucleotide variants in the marker genes
which leads to higher sensitivity. 

<img src="https://user-images.githubusercontent.com/6362936/128755895-90497fff-9342-459f-9891-28cd95f8f362.png" data-canonical-src="https://user-images.githubusercontent.com/6362936/128755895-90497fff-9342-459f-9891-28cd95f8f362.png" width="700" />

(via [the mOTUS website](https://motu-tool.org/))

# Installation and setup
If you are on OpenDemand, activate the pre-installed software with:
```
module use /gpfs/group/RISE/sw7/modules
module load anaconda
module load gcc/8.3.1
module load samtools
module unload python  #<<-- samtools auto-loads a different version of python than the one we want to use
conda activate bioconda  #<<-- alternatively: conda activate microbiome1
```

Otherwise, if you are on a different system, do a fresh install via:
```
conda create -y --name motus2 -c bioconda motus
conda activate motus2
```

## File structure and data:

Run the following commands to create the folder structure and download the necessary data:
```
cd ~
mkdir mOTUs_analysis
cd mOTUs_analysis
mkdir data scripts output
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```

# Create taxonomic profiles

mOTUs2 accepts as input short reads from a single shotgun
metagenomic sequencing experiment and outputs the list of detected
microbes and their relative abundances. 

## Input files

MetaPhlAn accepts metagenomic sequences in `.fastq`, `.sam`, or `.bam` formats.

mOTUs2 has a few different modes of operation, see these modes with `motus -h`:
```
Usage: motus <command> [options]

Command:
 -- Taxonomic profiling
      profile     Perform a taxonomic profiling (map_tax + calc_mgc + calc_motu)
      merge       Merge different taxonomic profiles to create a table

      map_tax     Map reads to the marker gene database
      calc_mgc    Aggregate reads from the same marker gene cluster (mgc)
      calc_motu   From a mgc abundance table, produce the mOTU abundance table

 -- SNV calling
      map_snv     Map reads to create bam/sam file for snv calling
      snv_call    SNV calling using metaSNV
```
We will first focus on the profiling ability of mOTUs: `motus profile -h`:
```
Usage: motus profile [options]

Input options:
   -f FILE[,FILE]  input file(s) for reads in forward orientation, fastq formatted
   -r FILE[,FILE]  input file(s) for reads in reverse orientation, fastq formatted
   -s FILE[,FILE]  input file(s) for reads without mate, fastq formatted
   -n STR          sample name
   -i FILE[,FILE]  provide sam or bam input file(s)
   -m FILE         provide a mgc reads count file

Output options:
   -o FILE         output file name [stdout]
   -I FILE         save the result of bwa in bam format (intermediate step) [None]
   -M FILE         save the mgc reads count (intermediate step) [None]
   -e              profile only reference species (ref_mOTUs)
   -c              print result as counts instead of relative abundances
   -p              print NCBI id
   -u              print the full name of the species
   -q              print the full rank taxonomy
   -B              print result in BIOM format
   -C STR          print result in CAMI format (BioBoxes format 0.9.1)
                   Values: [precision, recall, parenthesis]
   -k STR          taxonomic level [mOTU]
                   Values: [kingdom, phylum, class, order, family, genus, mOTU]

Algorithm options:
   -g INT          number of marker genes cutoff: 1=higher recall, 10=higher precision [3]
   -l INT          min. length of alignment for the reads (number of nucleotides) [75]
   -t INT          number of threads [1]
   -v INT          verbose level: 1=error, 2=warning, 3=message, 4+=debugging [3]
   -y STR          type of read counts [insert.scaled_counts]
                   Values: [base.coverage, insert.raw_counts, insert.scaled_counts]
```

For our purposes, the most important parameter is `-C` which specifies the CAMI format. There are three different choices:
```
-C precision    # higher precision
-C recall       # higher recall
-C parenthesis  # a balance between precision and recall
```

Since we now have practice running multiple samples, let's jump straight to:

## Run multiple samples

Let's run, in parallel, all the samples through mOTUs2. Put the following code in a file called `run_motus.sh`:
```
touch scripts/run_motus.sh
chmod +x scripts/run_motus.sh
nano scripts/run_motus.sh
```
Then paste the following into that file:
```
#!/usr/bin/env bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

scriptFolder=`dirname $0`  #<<-- where this script is located
baseFolder=$(dirname $scriptFolder)  #<<-- the main analysis folder (one up from the script folder)
outputFolder="${baseFolder}/output"

# Now analyze everything in one go
inputFolder="${baseFolder}/data"
for file in `ls ${inputFolder}/*.fastq`;
do
	baseName=$(basename $file)
	motus profile -s ${file} -o ${outputFolder}/${baseName%.fastq}.profile -t 5 -C parenthesis &
done
wait
exit
```
You can run it with:
```
./scripts/run_motus.sh
```

# Comparing differences between tools

Now that we have run both mOTUs2 and MetaPhlAn3 on the same data, we can compare the differences between these tools. Using TAMPA (which we installed previously) we can do this.
```
conda deactivate
conda activate bioconda  #<<-- or wherever you installed TAMPA
```
Let's pretend the mOTUs2 profile is the "ground truth" and compare its results on one sample to that of MetaPhlAn3:
```bash
sed -i 's/SampleID:.*/SampleID:Anterior_nares/g' ~/MetaPhlAn_analysis/output/SRS014464-Anterior_nares.cami_profile  #<<-- make the sample id the same for both tools
sed -i 's/SampleID:.*/SampleID:Anterior_nares/g' output/SRS014464-Anterior_nares.profile  #<<-- make the sample id the same for both tools
python /gpfs/group/RISE/sw7/anaconda/envs/bioconda/other/TAMPA/src/profile_to_plot.py -i ~/MetaPhlAn_analysis/output/SRS014464-Anterior_nares.cami_profile -g output/SRS014464-Anterior_nares.profile  -b output/mOTUs_vs_MetaPhlAn -nm genus
```
You should then see a file like the following in the folder `mOTUs_analysis/output`:
![mOTUs_vs_MetaPhlAn_tree_genus_Anterior_nares](https://user-images.githubusercontent.com/6362936/129259332-d8a45e9e-82ec-4e78-b8e8-ffa825620113.png)


So it looks like mOTUs2 found some Actinobacteria, while MetaPhlAn3 did not! Let's investigate why this is the case.

## Supporting evidence
let's chase down how mOTUs2 infered that Actinobacteris is present in the sample. Using the `-c` flag, we can indicate that we want read counts instead of relative 
abundances. We can also save the alignment BAM file with `-I` and the mOTU read counts with `-M` in the following fashion:

```bash
module load samtools  #<<-- see the funky issue with renaming python in the installation/activation section above
motus profile -s data/SRS014464-Anterior_nares.fastq -o output/SRS014464-Anterior_nares.motus_counts -I output/SRS014464-Anterior_nares.motus_bam -M output/SRS014464-Anterior_nares.motus_mgc -c
```

The `*.motus_counts` file contains the counts of _every_ mOTU in their database, so let's just select the ones with non-zero read counts:
```
grep -v '0$' output/SRS014464-Anterior_nares.motus_counts
```
Which gives an output of:
```
#consensus_taxonomy     unnamed sample
Corynebacterium pseudodiphtheriticum [ref_mOTU_v2_0478] 1
Dolosigranulum pigrum [ref_mOTU_v2_5089]        1
unknown Moraxella [meta_mOTU_v2_6740]   1
```
So it found a read that really did hit to Corynebacterium pseudodiphtheriticum. If you like, you can take a look at the alignment to investigate if this is spurious or not.

```bash
samtools view -h output/SRS014464-Anterior_nares.motus_bam > output/SRS014464-Anterior_nares.motus_sam
cd /gpfs/group/RISE/sw7/anaconda/anaconda3/envs/bioconda/share/motus-2.1.1/db_mOTU  #<<-- or wherever your installed version is
grep ref_mOTU_v2_0478 mOTU-LG.map.tsv | cut -f1 | cut -d'.' -f1 > ~/mOTUs_analysis/output/ref_mOTU_v2_0478.ids
cd ~/mOTUs_analysis/
grep -f output/ref_mOTU_v2_0478.ids output/SRS014464-Anterior_nares.motus_sam | grep -v '^@SQ'
 ```
 
 Then you can find exact matches in the SAM file such as:
 ```
 HWI-EAS109_102883399:3:86:17435:12622.lane0.single      16      refMG0011662.COG0172    95      0       100M <snip>
 ```
Meaning there's a 100bp match, so it really does look like Actinobacteria is in there!

# Please proceed to the [Assembly](Assembly.md) section
