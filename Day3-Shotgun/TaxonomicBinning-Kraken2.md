# Taxonomic binning with Kraken2
- [Taxonomic binning with Kraken2](#taxonomic-binning-with-kraken2)
  - [Setup and installation](#setup-and-installation)
    - [Folder structure](#folder-structure)
    - [Installing Kraken2](#installing-kraken2)
  - [Running Kraken2](#running-kraken2)
  - [Analyzing the output](#analyzing-the-output)
    - [Kraken2 on contigs](#kraken2-on-contigs)

Kraken is a very widely utilized metagenomics tool (dating back to 2014) and Kraken2 is its most recent iteration that boasts higher accuracy and decreased system resource utilization over Kraken1. 

In contrast to mOTUs2 and MetaPhlAn3 which use clade specific marker _genes_, Kraken2 utilizes clade specific marker _kmers_. This is summarized in the first figure of the [Kraken1 manuscript](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2014-15-3-r46):

![Kraken](https://user-images.githubusercontent.com/6362936/128582419-7da7594e-9483-4138-859e-d0f0e332bad3.PNG)

In using k-mers, Kraken2 has the ability to be blazingly fast, but longer query sequences are required to account for the inherent noise when utilizing k-mers in the size ranges that Kraken utilizes. Probably due to it's relatively early implementation and how fast the tool is, Kraken has been used in quite a few situation it should not have. For example, remember when we covered [taxonomic profiling](TaxonomicProfiling.md)? Well, taxonomic profiling doesn't seem _too_ different from taxonomic binning, right? A sequence is a sequence, right? Unfortunately, Kraken is quite sensitive to sequence length (as well as sequencing error). So many people were (incorrectly) using Kraken for taxonomic profiling that the authors of Kraken were motivated to publish [Bracken](https://peerj.com/articles/cs-104/) which is an extension of Kraken designed for taxonomic profiling. In [that manuscript](https://peerj.com/articles/cs-104/) the authors specifically state:
>Therefore, any assumption that Kraken’s raw read assignments can be directly translated into species- or strain-level abundance estimates (e.g.,  Schaeffer et al., 2015) is flawed, as ignoring reads at higher levels of the taxonomy will grossly underestimate some species, and creates the erroneous impression that Kraken’s assignments themselves were incorrect.

In the following, we will contrast the performance of Kraken when applied to raw reads and when applied to assembled contigs. But first, we need to set everything up!

## Setup and installation

### Folder structure
We are going to use out familiar folder structure and copy over both the raw reads as well as the assembled data corresponding to one of our samples:
```bash
cd ~
mkdir Kraken2_analysis
cd Kraken2_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB output/on_raw_reads
cp ~/MEGAHIT_analysis/output/default/final.contigs.fa data/MEGAHIT_default_contigs.fasta
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```

### Installing Kraken2

If you are using OnDemand, Kraken2 is already installed and you can activate it via:
```bash
module use /gpfs/group/RISE/sw7/modules
module load anaconda
conda deactivate
conda activate microbiome1
```

If you are on a different system, then Kraken2 is available on conda and is quite straightforward to install:
```
conda deactivate
conda create -y -n kraken2 kraken2
conda activate kraken2
```

After this, you need to choose what sort of database to use with Kraken. A mechanism is built in that downloads the default database (which is ~100GB in size), but alternate databases are available at: https://github.com/BenLangmead/aws-indexes/blob/master/docs/k2.md

We will be using the smallest database for the sake of time (it includes archaea, bacteria, viruses, plasmids, human sequences, and UniVec_Core (vector contamination). It is capped to only 8GB in size though, so shouldn't be used for serious research (I'd suggest using the default or the "PlusPF" database).

Even though this database is small-ish, it will likely make you exceed your home directory quota, so I've pre-downloaded the folder to the following location: `/gpfs/group/RISE/training/2021_microbiome/day5/k2train8gb`. We will make a symbolic link to this folder so that it looks like it's in the right place in your folder structure
```bash
ln -s /gpfs/group/RISE/training/2022_microbiome/k2train8gb data/
```

So please **_do not_** run the following command unless you direct it to some location where you have more disk quota available. The training data can be obtained and decompressed via:
```bash
wget https://genome-idx.s3.amazonaws.com/kraken/k2_standard_8gb_20210517.tar.gz -P data/k2train8gb
tar -xzvf data/k2train8gb/k2_standard_8gb_20210517.tar.gz -C data/k2train8gb
```
This will take some time to download.

## Running Kraken2

Running Kraken is quite straightforward (i.e. a single line of code), so we will jump directly to creating a script that will do all our analysis for us. First though, let's check out the Kraken parameters:
```
Usage: kraken2 [options] <filename(s)>

Options:
  --db NAME               Name for Kraken 2 DB
                          (default: none)
  --threads NUM           Number of threads (default: 1)
  --quick                 Quick operation (use first hit or hits)
  --unclassified-out FILENAME
                          Print unclassified sequences to filename
  --classified-out FILENAME
                          Print classified sequences to filename
  --output FILENAME       Print output to filename (default: stdout); "-" will
                          suppress normal output
  --confidence FLOAT      Confidence score threshold (default: 0.0); must be
                          in [0, 1].
  --minimum-base-quality NUM
                          Minimum base quality used in classification (def: 0,
                          only effective with FASTQ input).
  --report FILENAME       Print a report with aggregrate counts/clade to file
  --use-mpa-style         With --report, format report output like Kraken 1's
                          kraken-mpa-report
  --report-zero-counts    With --report, report counts for ALL taxa, even if
                          counts are zero
  --report-minimizer-data With --report, report minimizer and distinct minimizer
                          count information in addition to normal Kraken report
  --memory-mapping        Avoids loading database into RAM
  --paired                The filenames provided have paired-end reads
  --use-names             Print scientific names instead of just taxids
  --gzip-compressed       Input files are compressed with gzip
  --bzip2-compressed      Input files are compressed with bzip2
  --minimum-hit-groups NUM
                          Minimum number of hit groups (overlapping k-mers
                          sharing the same minimizer) needed to make a call
                          (default: 2)
  --help                  Print this message
  --version               Print version information
```
We will want to specify the database we downloaded with the `--db` flag, and save the following output files: `--classified-out` (the sequences that were classified along with their classification), `--output` (the normal output of Kraken2), and `--report` the summary of which clades were found in the input according to Kraken.

Let's go ahead and run Kraken on both assemblies, as well as the raw reads. We will put all of this in a script to execute everything in one go:
```bash
touch scripts/run_kraken.sh
chmod +x scripts/run_kraken.sh
nano scripts/run_kraken.sh
```
Then put the following into that file:
```
#!/usr/bin/env bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

scriptFolder=`dirname $0`  #<<-- where this script is located
baseDir=$(dirname $scriptFolder)  #<<-- the main analysis folder (one up from the script folder)
dataDir=${baseDir}/data
trainingDir=${dataDir}/k2train8gb
outputDir=${baseDir}/output
# The following is one way to do a for loop with pairs (similar to python's `zip` function): 
# I use a sequence of space-delimited strings like "A B" "C D". The for loop will take each string and split it up into an array. 
# Hence the loop args will look like:
# loop 1: args==["A" "B"]
# loop 2: args==["C" "D"] etc.
#for inOut in "MEGAHIT_default_contigs.fasta on_MEGAHIT" "GATB_default_contigs.fasta on_GATB" "SRS014464-Anterior_nares.fastq on_raw_reads";
for inOut in "MEGAHIT_default_contigs.fasta on_MEGAHIT" "SRS014464-Anterior_nares.fastq on_raw_reads";
do
  args=( $inOut );
  input=${dataDir}/${args[0]}
  output=${outputDir}/${args[1]}
  kraken2 kraken2 --db ${trainingDir} --threads 4 --output ${output}/kraken_default_output.txt --classified-out ${output}/kraken_classified_sequences.fq --use-names --report ${output}/kraken_report.txt ${input}
done
```
We can then run this script in the following fashion:
```bash
./scripts/run_kraken.sh
```

## Analyzing the output

The `kraken_classified_sequences.fq` files contain fastq formatted files of the reads/contigs with an NCBI taxID in the header of each sequence. This can be helpful for determining the correspondence between contigs and what organisms they originated from.

The `kraken_default_output.txt` contains a compressed-ish version of the above, but with additional information about what led to Kraken's inference. More details can be found [here](https://github.com/DerrickWood/kraken2/blob/master/docs/MANUAL.markdown#standard-kraken-output-format).

Lastly, the `kraken_report.txt` is what led to some interpreting Kraken2 as usable as a taxonomic profiler (it sure looks like a profile!). The first number is the percent of sequences that covered this taxon, followed by the number assigned to that clade, then to that taxon, a rank code, and then an NCBI taxID. More info can be found [here](https://github.com/DerrickWood/kraken2/blob/master/docs/MANUAL.markdown#sample-report-output-format) about the format.
Let's dig in a bit more:

### Kraken2 on contigs
Take a look at the `kraken_report.txt` in the `output/on_MEGAHIT` folder by using the command:
```bash
grep -w S output/on_MEGAHIT/kraken_report.txt | rev | cut -f1 |rev | sed 's/^ *//g'
```

```
Homo sapiens
Moraxella nonliquefaciens
```
See how this makes sense considering the BLAST results we saw earlier? In general, _this_ is what you want to be using instead of BLAST when you want to identify the taxa of each of your contigs. 


Now, let's contrast this to what happens when we look at the report in the raw reads directory: `output/on_raw_reads`.
```bash
grep -w S output/on_raw_reads/kraken_report.txt | rev | cut -f1 |rev | sed 's/^ *//g'
```
We get the following species:
```
Moraxella nonliquefaciens
Moraxella catarrhalis
Moraxella bovis
Moraxella bovoculi
Moraxella ovis
Pseudomonas tolaasii
Escherichia coli
Klebsiella aerogenes
Xanthomonas euvesicatoria
Haemophilus parainfluenzae
Methylophilus medardicus
Dolosigranulum pigrum
Streptococcus oralis
Staphylococcus epidermidis
Staphylococcus sp. SB1-57
Staphylococcus warneri
Paenibacillus kribbensis
Finegoldia magna
Fastidiosipila sanguinis
Cutibacterium acnes
Cutibacterium avidum
Corynebacterium propinquum
Corynebacterium segmentosum
Corynebacterium macginleyi
Corynebacterium sp. FDAARGOS 1242
Corynebacterium sp. LMM-1652
Corynebacterium kefirresidentii
Corynebacterium stationis
Corynebacterium glucuronolyticum
Corynebacterium epidermidicanis
Corynebacterium kutscheri
Corynebacterium diphtheriae
Corynebacterium xerosis
Lawsonella clevelandensis
Streptomyces tsukubensis
Prevotella denticola
Homo sapiens
Propionibacterium virus Ouroboros
Propionibacterium virus PHL041M10
```
This is what the Braken paper was refering to when it said not to trust the species classifications. Of course one could take a look higher ranks, like the Genus level:
```
Moraxella
Pseudomonas
Escherichia
Klebsiella
Xanthomonas
Haemophilus
Pseudoalteromonas
Neisseria
Methylophilus
Sphingobium
Dolosigranulum
Streptococcus
Staphylococcus
Paenibacillus
Finegoldia
Fastidiosipila
Cutibacterium
Corynebacterium
Lawsonella
Streptomyces
Bacteroides
Prevotella
Homo
Pahexavirus
```
but here again we see lots of false positives. One is left to determine on their own what is an acceptable threshold of "% reads classified." Moral of the story? Use Kraken for taxonomic binning of contigs, use Braken (or MetaPhlAn3, or mOTUs2) for taxonomic profiling.

# Congrats on finishing the "intro to WGS metagenomics" tutorial!
