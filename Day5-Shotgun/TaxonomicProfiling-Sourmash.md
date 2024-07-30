
**Sourmash 3.0 Tutorial**


# Installation

## Activation on OpenDemand
If you are taking part in the workshop, use the following commands to activate the environment.
```
module use /gpfs/group/RISE/sw7/modules
module load anaconda
conda activate microbiome2
```

## Full installation
This instructions are _only_ if you would like to install it with conda elsewhere (eg. on the open queue, a personal computer, etc.)
The basic steps are: load conda, create the environment, then install the tool:
```bash
module load anaconda3
conda create -y --name metaphlan -c bioconda python=3.7 tbb=2020.2 metaphlan
conda activate metaphlan
```

## Set up directories and obtain the test data:
- Make analysis folders:
```
cd ~
mkdir MetaPhlAn_analysis  #<<-- main analysis folder
cd MetaPhlAn_analysis  #<<-- go inside this folder
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

Each of you will get a random metagenomic sample and with the help of this tutorial, it is your quest to report the sample that you were tasked to study!

# Introduction

As sequencing methods become more inexpensive, producing genomic and metagenomic data becomes more available for 
large-scale analyses. Therefore, the development of quick and accuarate computational approaches become essential. Here, we introduce one such program, sourmash, which enables the minimization of sequencing data, similarity comparisons, species identification, among other tasks [[1]](#1).

## FracMinHash Sketch 

Utilizing sourmash, we can sketch a simpler representation of the original genomic or metagenomic datasets using the FracMinHash approach. 

![alt text](https://github.com/bioinfwithjudith/sourmash_tutorial/blob/main/src/Picture3.png)

Simply put it, a minimized set of k-mers (a subset of a string of length k) is produced from a fasta file with sequences of interest. The threshold to which the set is minimized is indicated by a scale factor, a parameter used to reduce the original set. Note that if a scale factor of 1 is utilized, the final FracMinHash sketch will be the original set of k-mers and as this number increases, the more the original set of k-mers will decrease for the final FracMinHash sketch. Each k-mer in the original set is passed through a Hash function, k-mers that minimize this function by having a value that is equal or less to the scale factor are used in the final FracMinHash sketch, FRAC(A) for example. 

![alt text](https://github.com/bioinfwithjudith/sourmash_tutorial/blob/main/src/Picture6.png)

## Comparing similarity of two FracMinHash sketches 

One of the beneficial tasks of sourmash is to estimate similarity between sketches offering the similarity indexes Jaccard and Containment. 

**Jaccard.** The Jaccard index is one of an earlier similarity index used to estimate the size of the union of two sets. However, as the sizes between two sets becomes more dissimilar, the less reliable is this estimation[[2]](#2).

**Containment.** The Containment index addresses this issue by estimating what is contained from one Set in another so the differing size of two sets is not an issue[[2]](#2).

![alt text](https://github.com/bioinfwithjudith/sourmash_tutorial/blob/main/src/Picture8.png)


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

## Search an md5 identifier and report similarity

Let's revisit the signature file that was produced using the `--singleton` flag. A manifest file contains md5 identifiers for each sketch within a signature file. To create your manifest file, use the following command:

`sourmash sig manifest sample_001.fna.singleton.sig -o sample_001.manifest.csv`

|Parameter      |Description|
|---------------|----------|
|sourmash sig manifest| The command that sourmash utilizes to produce a manifest file. |
|sample_001.fna.singleton.sig       | signature filename 1 that includes a sketch for each sequence. |
|-o           | Output name of manifest file. The output name can also be indicated using `--csv` |


If you open the newly produced manifest file, you'll observe that each sketch in your signature file has a md5 identifier. For tutorial purposes, I only show the first few lines.

|internal_location|md5|md5short|ksize|moltype|num|scaled|n_hashes|with_abundance|name|filename|
|-|-|-|-|-|-|-|-|-|-|-|
sample_001.fna.sig|d3513280b35b2a918a7181875c0683c8|d3513280|31|DNA|0|500|22|0|genome_A_sequence_A|datasets/sample_001.fna|
sample_001.fna.sig|2d96ee330b6b295a06b70fdbfb75af34|2d96ee33|31|DNA|0|500|21|0|genome_A_sequence_0|datasets/sample_001.fna|
sample_001.fna.sig|364c20a1ca43d3ae3e9ae7d9e9a6c837|364c20a1|31|DNA|0|500|19|0|genome_B_sequence_0|datasets/sample_001.fna|

If we copy the first `md5` identifer in this file, we can `sourmash search` how much of this sequence is in **sample_002.fna.singleton.sig**

`sourmash search sample_001.fna.singleton.sig sample_002.fna.singleton.sig --md5 d3513280b35b2a918a7181875c0683c8 --containment`

The md5 identifier that we are interested in is found across three sequences in sample_002.fna.singleton.sig with the following percentages.

```
== This is sourmash version 4.8.6. ==
== Please cite Brown and Irber (2016), doi:10.21105/joss.00027. ==

select query k=31 automatically.
loaded query: genome_A_sequence_A... (k=31, DNA)
--
loaded 10 total signatures from 1 locations.
after selecting signatures compatible with search, 10 remain.

9 matches above threshold 0.080; showing first 3:
similarity   match
----------   -----
100.0%       ref_gene
 90.9%       genome_A_2
 86.4%       genome_A_1
WARNING: size estimation for at least one of these sketches may be inaccurate. ANI values will not be reported for these comparisons.
```
# Report abundance using sourmash gather

One of the many tasks in metagenomic research, is reporting the abundanes of species within a sample. We can accomplish this utilizing `sourmash gather`.

<span style="color:red">Unsure if this is working, running `sourmash gather --help` returning error</span>.



# References
<a id="1">[1]</a> 
Brown, C. T., & Irber, L. (2016). sourmash: a library for MinHash sketching of DNA. Journal of open source software, 1(5), 27.

<a id="2">[2]</a> 
Koslicki, D., & Zabeti, H. (2019). Improving minhash via the containment index with applications to metagenomic analysis. Applied Mathematics and Computation, 354, 206-215.

<a id="3">[3]</a> 
Hera, M. R., Pierce-Ward, N. T., & Koslicki, D. (2023). Deriving confidence intervals for mutation rates across a wide range of evolutionary distances using FracMinHash. Genome research, 33(7), 1061-1068.

# Please proceed to the [mOTUs2 Section](TaxonomicProfiling-mOTUs2.md)
