
**Sourmash 3.0 Tutorial**


# Installation

## Activation on OpenDemand
If you are taking part in the workshop, use the following commands to activate the environment.
```
module load anaconda
conda activate /storage/icds/RISE/training/microbiome/2024/envs/microbiome
```

## Full installation
This instructions are _only_ if you would like to install it with conda elsewhere (eg. on the open queue, a personal computer, etc.)
The basic steps are: load conda, create the environment, then install the tool:
```bash
module load anaconda3
conda create -y --name sourmash install bioconda::sourmash
conda activate sourmash
```

## Set up directories and obtain the test data:
- Make analysis folders:
```
cd ~
mkdir sourmash_analysis  #<<-- main analysis folder
cd sourmash_analysis  #<<-- go inside this folder
mkdir data output scripts  #<<-- make three directories: data, output, and scripts
```
Then download the data

```
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2022/main/Day5-Shotgun/Data/file_list.txt  #<<-- downloads the data from github
ls *.gz | xargs -P6 -I{} gunzip {}  #<<-- decompresses the data in parallel
cd ..  #<<-- move back up a directory
```
# The MetaGenomQuest Tutorial
(Courtesy of Judith Rodriguez: https://github.com/bioinfwithjudith/sourmash_tutorial)

Each of you will get a random metagenomic sample and with the help of this tutorial, it is your quest to report the sample that you were tasked to study!

# Introduction

As sequencing methods become more inexpensive, producing genomic and metagenomic data becomes more available for 
large-scale analyses. Therefore, the development of quick and accuarate computational approaches become essential. Here, we introduce one such program, sourmash, which enables the minimization of sequencing data, similarity comparisons, species identification, among other tasks [[1]](#1).

## FracMinHash Sketch 

Utilizing sourmash, we can sketch a simpler representation of the original genomic or metagenomic datasets using the FracMinHash approach. 

![alt text](Data/Picture3.png)

Simply put it, a minimized set of k-mers (a subset of a string of length k) is produced from a fasta file with sequences of interest. The threshold to which the set is minimized is indicated by a scale factor, a parameter used to reduce the original set. Note that if a scale factor of 1 is utilized, the final FracMinHash sketch will be the original set of k-mers and as this number increases, the more the original set of k-mers will decrease for the final FracMinHash sketch. Each k-mer in the original set is passed through a Hash function, k-mers that minimize this function by having a value that is equal or less to the scale factor are used in the final FracMinHash sketch, FRAC(A) for example. 

![alt text](Data/Picture6.png)

## Comparing similarity of two FracMinHash sketches 

One of the beneficial tasks of sourmash is to estimate similarity between sketches offering the similarity indexes Jaccard and Containment. 

**Jaccard.** The Jaccard index is one of an earlier similarity index used to estimate the size of the union of two sets. However, as the sizes between two sets becomes more dissimilar, the less reliable is this estimation[[2]](#2).

**Containment.** The Containment index addresses this issue by estimating what is contained from one Set in another so the differing size of two sets is not an issue[[2]](#2).

![alt text](Data/Picture8.png)


# Preprocessing data utilizing sourmash sketch

To start our sourmash tutorial, we will sketch our fasta file of interest: **sample_001.fna**. This file does not contain any meaningfull information but is just a play dataset for tutorial purposes. 

`sourmash sketch dna datasets/sample_001.fna -p k=31,scaled=500`

|Parameter      |Description|
|---------------|----------|
|sourmash sketch| The command that sourmash utilizes to produce a sketch. |
|dna            | Identify that the sequences in our X file are DNA. If a different sequence type is used, like protein sequences, then this would be indicated as **protein** instead.|
|sample_001.fna       | Filename of interest. Please note that sourmash can produce sketches from either FASTA or FASTQ fules.|
|-p           | Flag to indicate a list of parameters. |
|k=33           | Tunable parameter required by the user to set. Check how the ksize can affect further analyses here: (CITE). |
|scaled=500     | Our scale factor, which is also tunable.|

Utilizing sourmash sketch, we have produced the following  file, known as a signature file: **sample_001.fna.sig**. 

Let's be a bit anosey and see what this file contains:

`sourmash sig describe sample_001.fna.sig`

Below is the information you should see displayed on your end. Information such as the signature filename, sequence type, ksize and scale factor used are report. Additional information, such as total number of signatures and hashes are also shown, alongside other information.

```
== This is sourmash version 4.8.6. ==
== Please cite Brown and Irber (2016), doi:10.21105/joss.00027. ==

** loading from 'sample_001.fna.sig'
path filetype: MultiIndex
location: sample_001.fna.sig
is database? no
has manifest? yes
num signatures: 1
** examining manifest...
total hashes: 21
summary of sketches:
   1 sketches with DNA, k=31, scaled=1000             21 total hashes
```

# Comparing and Searching signatures

## Compare similarity of two sketches with sourmash compare

Now that we feel a bit more comfortable with a sketching fasta file. Let's sketch a another file to compare it to the previous signature file we produced. Notice, that we are using the same ksize and scale factor, sourmash requires that two signatures have parameters tuned in the same way for comparison. 

`sourmash sketch dna datasets/sample_002.fna -p k=31,scaled=500`

Let's estimate the containment index of our signature files utilizing `sourmash compare`!

`sourmash compare sample_001.fna.sig sample_002.fna.sig --containment`

You should see the following output, where we have the names of the original fasta files and the containment between these files.

```
== This is sourmash version 4.8.6. ==
== Please cite Brown and Irber (2016), doi:10.21105/joss.00027. ==

loaded 2 signatures total.
NOTE: downsampling to scaled value of 1000

0-sample_001.fna        [1.    0.696]
1-sample_002.fna        [0.762 1.   ]
min similarity in matrix: 0.696
WARNING: size estimation for at least one of these sketches may be inaccurate. ANI values will be set to 1 for these comparisons.
```

These results can be reported to a csv file for further analyses.

` sourmash compare sample_001.fna.sig sample_002.fna.sig --containment --csv compare.csv`

|Parameter      |Description|
|---------------|----------|
|sourmash compare| The command that sourmash utilizes to compare two signatures. |
|sample_001.fna.sig       | signature filename 1. |
|sample_002.fna.sig       | signature filename 2. |
|--containment           | Flag to indicate we are using the similarity index containment. Other similarity indexes that can be used are **--jaccard** or **--ani** |
|--csv           | Flag to indicate we want to produce a CSV file to report a matrix with similarity indexes |

Looking at our newly produced csv file, we would expect the following similarity matrix:

```
sample_001.fna,sample_002.fna
1.0,0.6956521739836137
0.7619047624764425,1.0
```

## Compare individual sequences between signatures.

As you may have noticed, the comparisons reported are between two fasta files as a whole. However, you might be interested in comparing the sequences individually than the sequences as a whole. To accomplish this, we revisit `sourmash sketch`. 

Let's modify our `sourmaash sketch` command by adding the `--singleton` flag, this will produce a signature file where each sequence is sketched individually.

`sourmash sketch dna datasets/sample_001.fna --singleton -p k=31,scaled=500 -o sample_001.fna.singleton.sig`

Compare the description between the first signature file we produced in which the `--singleton` flag was not utilized (**sample_001.fna.sig**) ato our new siganture file that does utilize the `--singleton` flag, **sample_001.fna.singleton.sig**.

`sourmash sig fileinfo sample_001.fna.singleton.sig`

Instead of having just one sketch representing the fasta file as a whole, the `--singleton` flag produces a signature file with 10 sketches representing the 10 sequences within our fasta file of interest. 

```

== This is sourmash version 4.8.6. ==
== Please cite Brown and Irber (2016), doi:10.21105/joss.00027. ==

** loading from 'test.sig'
path filetype: MultiIndex
location: sample_001.fna.singleton.sig
is database? no
has manifest? yes
num signatures: 10
** examining manifest...
total hashes: 210
summary of sketches:
   10 sketches with DNA, k=31, scaled=500             210 total hashes
```

We can run the same command for coomparing two signature files, and the csv file produced will contain the matrix for individual sequences.

`sourmash compare sample_001.fna.singleton.sig sample_002.fna.singleton.sig --containment --csv compare.singleton.sig.csv`

## Search and report overall similarity percentages using sourmash search

Maybe you would like to report the percent of how much there is of one sample in another sample.

`sourmash search sample_001.fna.sig sample_002.fna.sig --containment`

|Parameter      |Description|
|---------------|----------|
|sourmash search| The command that sourmash utilizes to search how much a query is in a sample. |
|sample_001.fna.sig       | signature filename 1. |
|sample_002.fna.sig       | signature filename 2. |
|--containment           | Flag to indicate we are using the similarity index containment. Other similarity indexes that can be used are `--jaccard` |

According to the report below, there is ~76% of **sample_001.fna.sig** in **sample_002.fna.sig**.

```
== This is sourmash version 4.8.6. ==
== Please cite Brown and Irber (2016), doi:10.21105/joss.00027. ==

select query k=31 automatically.
loaded query: sample_001.fna... (k=31, DNA)
--
loaded 1 total signatures from 1 locations.
after selecting signatures compatible with search, 1 remain.

1 matches above threshold 0.080:
similarity   match
----------   -----
 76.2%       sample_002.fna
 ```


# Other example uses
(courtesy of the DIB lab: https://sourmash.readthedocs.io/en/latest/tutorial-basic.html)

Download some reads and a reference genome:

```
mkdir ~/data
cd ~/data
curl -L https://osf.io/ruanf/download -o ecoliMG1655.fa.gz
curl -L https://osf.io/q472x/download -o ecoli_ref-5m.fastq.gz
```

Compute a scaled signature from our reads:

```
mkdir ~/sourmash
cd ~/sourmash

sourmash sketch dna -p scaled=10000,k=31 ~/data/ecoli_ref*.fastq.gz -o ecoli-reads.sig
```

## Compare reads to assemblies

Use case: how much of the read content is contained in the reference genome?

Build a signature for an E. coli genome:

```
sourmash sketch dna -p scaled=1000,k=31 ~/data/ecoliMG1655.fa.gz -o ecoli-genome.sig
```

and now evaluate *containment*, that is, what fraction of the read content is
contained in the genome:

```
sourmash search ecoli-reads.sig ecoli-genome.sig --containment
```

and you should see:

```

select query k=31 automatically.
loaded query: /home/jovyan/data/ecoli_ref-5m... (k=31, DNA)
loaded 1 signatures.

1 matches:
similarity   match
----------   -----
 31.0%       /home/jovyan/data/ecoliMG1655.fa.gz
```


Try the reverse, too!

```
sourmash search ecoli-genome.sig ecoli-reads.sig --containment
```

## Make and search a database quickly.

Suppose that we have a collection of signatures (made with `sourmash
compute` as above) and we want to search it with our newly assembled
genome (or the reads, even!). How would we do that?

Let's grab a sample collection of 50 E. coli genomes and unpack it --

```
mkdir ecoli_many_sigs
cd ecoli_many_sigs

curl -O -L https://github.com/sourmash-bio/sourmash/raw/latest/data/eschericia-sigs.tar.gz

tar xzf eschericia-sigs.tar.gz
rm eschericia-sigs.tar.gz

cd ../

```

This will produce 50 files named `ecoli-N.sig` in the directory `ecoli_many_sigs/` --

```
ls ecoli_many_sigs
```

Let's turn this into an easily-searchable database with `sourmash index` --

```
sourmash index ecolidb ecoli_many_sigs/*.sig
```

and now we can search!

```
sourmash search ecoli-genome.sig ecolidb.sbt.zip -n 20
```

You should see output like this:

```
select query k=31 automatically.
loaded query: /home/ubuntu/data/ecoliMG1655.... (k=31, DNA)
loaded 0 signatures and 1 databases total.                                     

49 matches; showing first 20:
similarity   match
----------   -----
 75.9%       NZ_JMGW01000001.1 Escherichia coli 1-176-05_S4_C2 e117605...
 73.0%       NZ_JHRU01000001.1 Escherichia coli strain 100854 100854_1...
 71.9%       NZ_GG774190.1 Escherichia coli MS 196-1 Scfld2538, whole ...
 70.5%       NZ_JMGU01000001.1 Escherichia coli 2-011-08_S3_C2 e201108...
 69.8%       NZ_JH659569.1 Escherichia coli M919 supercont2.1, whole g...
 59.9%       NZ_JNLZ01000001.1 Escherichia coli 3-105-05_S1_C1 e310505...
 58.3%       NZ_JHDG01000001.1 Escherichia coli 1-176-05_S3_C1 e117605...
 56.5%       NZ_MIWF01000001.1 Escherichia coli strain AF7759-1 contig...
 56.1%       NZ_MOJK01000001.1 Escherichia coli strain 469 Cleandata-B...
 56.1%       NZ_MOGK01000001.1 Escherichia coli strain 676 BN4_676_1_(...
 50.5%       NZ_KE700241.1 Escherichia coli HVH 147 (4-5893887) acYxy-...
 50.3%       NZ_APWY01000001.1 Escherichia coli 178200 gec178200.conti...
 48.8%       NZ_LVOV01000001.1 Escherichia coli strain swine72 swine72...
 48.8%       NZ_MIWP01000001.1 Escherichia coli strain K6412 contig_00...
 48.7%       NZ_AIGC01000068.1 Escherichia coli DEC7C gecDEC7C.contig....
 48.2%       NZ_LQWB01000001.1 Escherichia coli strain GN03624 GCID_EC...
 48.0%       NZ_CCQJ01000001.1 Escherichia coli strain E. coli, whole ...
 47.3%       NZ_JHMG01000001.1 Escherichia coli O121:H19 str. 2010EL10...
 47.2%       NZ_JHGJ01000001.1 Escherichia coli O45:H2 str. 2009C-4780...
 46.5%       NZ_JHHE01000001.1 Escherichia coli O103:H2 str. 2009C-327...

```

## Compare many signatures and build a tree.

Compare all the things:

```
sourmash compare ecoli_many_sigs/* -o ecoli_cmp
```

Optionally, parallelize to 8 threads using `-p 8`:

```
sourmash compare -p 8 ecoli_many_sigs/* -o ecoli_cmp
```

and then plot:

```
sourmash plot --pdf --labels ecoli_cmp
```

which will produce files named `ecoli_cmp.matrix.pdf` and
`ecoli_cmp.dendro.pdf`.

Here's a PNG version:

![E. coli comparison plot](Data/ecoli_cmp.png)


# References
<a id="1">[1]</a> 
Irber, L., et al. (2024). sourmash v4: A multitool to quickly search, compare, and analyze genomic and metagenomic data 
sets. Journal of Open Source Software, 9(98), 6830. https://doi.org/10.21105/joss.06830

<a id="2">[2]</a> 
Koslicki, D., & Zabeti, H. (2019). Improving minhash via the containment index with applications to metagenomic analysis. Applied Mathematics and Computation, 354, 206-215.

<a id="3">[3]</a> 
Hera, M. R., Pierce-Ward, N. T., & Koslicki, D. (2023). Deriving confidence intervals for mutation rates across a wide range of evolutionary distances using FracMinHash. Genome research, 33(7), 1061-1068.

# Please proceed to the [YACHT Section](TaxonomicProfiling-YACHT.md)
