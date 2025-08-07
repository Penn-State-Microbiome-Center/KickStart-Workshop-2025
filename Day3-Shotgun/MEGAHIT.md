# MEGAHIT
- [MEGAHIT](#megahit)
  - [Installation and setup](#installation-and-setup)
  - [Usage](#usage)
    - [File directory setup and data acquisition](#file-directory-setup-and-data-acquisition)
    - [Running MEGAHIT](#running-megahit)
  - [Analyzing the assembly](#analyzing-the-assembly)
    - [Installing QUAST](#installing-quast)
    - [Running QUAST](#running-quast)

MEGAHIT is a _de novo_ assembler first introduced in 2015 which has been continually updated since then. It utilizes mutliple k-mer sizes when building a de Bruijn graph 
along with a strategy to rescue low-coverage regions while attempting to account for sequencing error.

From [the MEGAHIT publication](https://academic.oup.com/bioinformatics/article/31/10/1674/177884):
![btv033f1p](https://user-images.githubusercontent.com/6362936/128756606-090fa301-30df-49b9-83a0-ed72482925d7.gif)


## Installation and setup

### File directory setup and data acquisition 
Let's make a top-level directory for our analysis, the sub-directories, and download the data as usual:
```
cd ~
mkdir MEGAHIT_analysis
cd MEGAHIT_analysis
mkdir data scripts output
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2022/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```


MEGAHIT is already installed in the OnDemand environment, and you can activate it via:
```
module use /gpfs/group/RISE/sw7/modules
module load anaconda
conda deactivate
conda activate microbiome1
```

Otherwise, if you are on a different system, MEGAHIT can be installed with conda in the following fashion:
```
conda create -y -n megahit -c bioconda megahit
conda activate megahit
```

## Usage
There are a variety of parameters that can be specified with MEGAHIT:
```
megahit: MEGAHIT v1.2.9

contact: Dinghua Li <voutcn@gmail.com>

Usage:
  megahit [options] {-1 <pe1> -2 <pe2> | --12 <pe12> | -r <se>} [-o <out_dir>]

  Input options that can be specified for multiple times (supporting plain text and gz/bz2 extensions)
    -1                       <pe1>          comma-separated list of fasta/q paired-end #1 files, paired with files in <pe2>
    -2                       <pe2>          comma-separated list of fasta/q paired-end #2 files, paired with files in <pe1>
    --12                     <pe12>         comma-separated list of interleaved fasta/q paired-end files
    -r/--read                <se>           comma-separated list of fasta/q single-end files

Optional Arguments:
  Basic assembly options:
    --min-count              <int>          minimum multiplicity for filtering (k_min+1)-mers [2]
    --k-list                 <int,int,..>   comma-separated list of kmer size
                                            all must be odd, in the range 15-255, increment <= 28)
                                            [21,29,39,59,79,99,119,141]

  Another way to set --k-list (overrides --k-list if one of them set):
    --k-min                  <int>          minimum kmer size (<= 255), must be odd number [21]
    --k-max                  <int>          maximum kmer size (<= 255), must be odd number [141]
    --k-step                 <int>          increment of kmer size of each iteration (<= 28), must be even number [12]

  Advanced assembly options:
    --no-mercy                              do not add mercy kmers
    --bubble-level           <int>          intensity of bubble merging (0-2), 0 to disable [2]
    --merge-level            <l,s>          merge complex bubbles of length <= l*kmer_size and similarity >= s [20,0.95]
    --prune-level            <int>          strength of low depth pruning (0-3) [2]
    --prune-depth            <int>          remove unitigs with avg kmer depth less than this value [2]
    --disconnect-ratio       <float>        disconnect unitigs if its depth is less than this ratio times
                                            the total depth of itself and its siblings [0.1]
    --low-local-ratio        <float>        remove unitigs if its depth is less than this ratio times
                                            the average depth of the neighborhoods [0.2]
    --max-tip-len            <int>          remove tips less than this value [2*k]
    --cleaning-rounds        <int>          number of rounds for graph cleanning [5]
    --no-local                              disable local assembly
    --kmin-1pass                            use 1pass mode to build SdBG of k_min

  Presets parameters:
    --presets                <str>          override a group of parameters; possible values:
                                            meta-sensitive: '--min-count 1 --k-list 21,29,39,49,...,129,141'
                                            meta-large: '--k-min 27 --k-max 127 --k-step 10'
                                            (large & complex metagenomes, like soil)

  Hardware options:
    -m/--memory              <float>        max memory in byte to be used in SdBG construction
                                            (if set between 0-1, fraction of the machine's total memory) [0.9]
    --mem-flag               <int>          SdBG builder memory mode. 0: minimum; 1: moderate;
                                            others: use all memory specified by '-m/--memory' [1]
    -t/--num-cpu-threads     <int>          number of CPU threads [# of logical processors]
    --no-hw-accel                           run MEGAHIT without BMI2 and POPCNT hardware instructions

  Output options:
    -o/--out-dir             <string>       output directory [./megahit_out]
    --out-prefix             <string>       output prefix (the contig file will be OUT_DIR/OUT_PREFIX.contigs.fa)
    --min-contig-len         <int>          minimum length of contigs to output [200]
    --keep-tmp-files                        keep all temporary files
    --tmp-dir                <string>       set temp directory

Other Arguments:
    --continue                              continue a MEGAHIT run from its last available check point.
                                            please set the output directory correctly when using this option.
    --test                                  run MEGAHIT on a toy test dataset
    -h/--help                               print the usage message
    -v/--version                            print version
```
However, the main ones we will be concerned with are specifying if the input data (which must be `fasta` or `fastq`) is paired end (`-1` and `-2` flags), interleaved (`-12` flag) 
or `-r` single-end, as well as the specification of the output directory with `-o`.

### Running MEGAHIT

To run MEGAHIT using the default parameters on one of the samples we have, all it takes is the following:
```
megahit -r data/SRS014464-Anterior_nares.fasta -o output/default
```

Before we investigate the output of MEGAHIT, let's run it with a few different parameter settings.

We could specify that _all_ k-mers should be considered, and can indicate this by setting the minimum k-mer count to 1:
```
megahit -r data/SRS014464-Anterior_nares.fasta -o output/min1 --min-count 1
```
This will likely result in many more, shorter contigs due to trusting _every_ k-mer as a true k-mer. I.e. Since we have ignored the effect of noise, we will likely 
have a variety of contigs that only differ by a few bases, which are likely due to sequencing error.

Alternatively, we could also change what range of k-mer sizes we want to use. In general, the larger the k-mer size you use, the more specific (and less sensitive) your assemblies will be. 
In practice, this _can_ result in the assembly of high abundance organisms. Inversely, the smaller the k-mer size, the more sensitive (but less specific) your assembly will be. 
I.e. you may get a bunch of really short contigs.

```
megahit -r data/SRS014464-Anterior_nares.fasta -o output/ksize15-51-10 --k-min 15 --k-max 51 --k-step 10
```

## Analyzing the assembly

Now that we've assembled one of these samples, let's go ahead and run some basic statistics on it. [QUAST](http://bioinf.spbau.ru/quast) is the premier genomic and metagenomic 
assembly assessment package. If you provide it a reference, it can can provide you quite a bit of insightful information as to the quality of your assembly. Even without 
a reference though, you can still use it to gain some insight into your assembly.

### Installing QUAST

QUAST comes pre-installed in the OnDemand system and can be activated via:
```bash
conda deactivate
module use /gpfs/group/RISE/sw7/modules
module load anaconda
conda activate microbiome2
```

If you are on a different system, to install QUAST, we simply need to run:
```
conda deactivate
conda create --prefix ~/quast -y -c bioconda quast bwa bedtools
conda activate ~/quast
```

### Running QUAST
There are a number of different ways to run QUAST, but since we are doing _de novo_ metagenome assembly, we do not have a reference to compare to. As such, we can just run QUAST 
to get some insight about the basic assembly quality.

MEGAHIT returns a main `final.contigs.fa` in the output directory which contains the assembled contigs. As such, we can run quast on a single assembly in the following way:
```
quast -o output/default/quast_out `#<<-- specify the output folder` \
 -m 250 `#<<-- consider 250bp+ as a contig` \
 --circos `#<<-- make a circos plot` \
 --glimmer `#<<-- use glimmer to predict genes (note: you should use -mgm (MetaGeneMark) instead, but we would need everyone to agree to a license and download it, so we aren't doing that here` \
 --rna-finding `#<<-- try to find ribosomal RNA genes` \
 --single data/SRS014464-Anterior_nares.fasta `#<-- tell QUAST where the reads were so you can map it back to the assembly` \
  output/default/final.contigs.fa `#<<-- the input assembly`
 
```
Note: running this in a single line without the comments looks like:
```
quast -o output/default/quast_out -m 250 --circos --glimmer --rna-finding --single data/SRS014464-Anterior_nares.fasta output/default/final.contigs.fa
```

If you download the QUAST report, you can view it in a browser by clicking the `report.html` tag. It should look something like this:
![quast](https://user-images.githubusercontent.com/6362936/128277768-bb09a609-6a67-4997-8812-12202d19ec19.PNG)


Let's go ahead and run QUAST on each of the assemblies. We will create a file that will automatically create a QUAST report for each of the assemblies.
```
touch scripts/run_quast.sh
chmod +x scripts/run_quast.sh
nano scripts/run_quast.sh  #<<-- or vim or the like
```
Then paste the following into that file:
```
#!/usr/bin/env bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

scriptFolder=`dirname $0`  #<<-- where this script is located
baseFolder=$(dirname $scriptFolder)  #<<-- the main analysis folder (one up from the script folder)
outputFolder="${baseFolder}/output"  #<<-- output folder
dataFolder="${baseFolder}/data"  #<<-- input folder

# Now analyze everything in one go
for folder in `ls -d ${outputFolder}/*`;
do
        quast -o ${folder}/quast_out -m 250 --circos --glimmer --rna-finding --single ${dataFolder}/SRS014464-Anterior_nares.fasta ${folder}/final.contigs.fa
done

```
Then run it:
```
./scripts/run_quast.sh
```

Now that we have run all the analyses, let's compare their results by downloading the associated output folders and viewing the html output.
**Can you tell which of the assemblies was the best from viewing the reports?**

**In which situations would the different MEGAHIT parameter settings be useful?**

# Please proceed to the [GATB](GATB.md) section
