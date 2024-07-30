
**MetaPhlAn 3.0 Tutorial**
===========================
Borrowing heavily from [here](https://github.com/biobakery/biobakery/wiki/metaphlan3).
- [Installation](#installation)
  - [Activation on OpenDemand](#activation-on-opendemand)
  - [Full installation](#full-installation)
  - [Set up directories and obtain the test data:](#set-up-directories-and-obtain-the-test-data)
- [**Overview**](#overview)
  - [**Create taxonomic profiles**](#create-taxonomic-profiles)
    - [**Input files**](#input-files)
    - [**Run a single sample**](#run-a-single-sample)
    - [**Output files**](#output-files)
    - [**bowtie2out file as input**](#bowtie2out-file-as-input)
    - [**Run multiple samples**](#run-multiple-samples)
    - [**Merge outputs**](#merge-outputs)
  - [**Visualize results**](#visualize-results)
    - [**Simple Vizualization with TAMPA**](#simple-vizualization-with-tampa)
    - [**Create a heatmap with hclust2**](#create-a-heatmap-with-hclust2)
    - [**Create a cladogram with GraPhlAn**](#create-a-cladogram-with-graphlan)

[MetaPhlAn](https://github.com/biobakery/MetaPhlAn/tree/3.0) is a tool
for profiling the taxonomic composition of microbial communities from metagenomic
shotgun sequencing data.

This tool is quite fast and has high specificity, but sacrifices sensitivity. It is based on clade specific marker genes.

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

------------------------------------------------------------------------
# **Overview**
----------------

The basic steps of MetaPhlAn are:

[![MetaPhlAn2.png](https://github.com/biobakery/biobakery/blob/master/images/2526749054-MetaPhlAn2.png)](https://github.com/biobakery/biobakery/blob/master/images/2526749054-MetaPhlAn2.png)

------------------------------------------------------------------------
## **Create taxonomic profiles**
-----------------------------

MetaPhlAn accepts as input short reads from a single shotgun
metagenomic sequencing experiment and outputs the list of detected
microbes and their relative abundances.
### **Input files**

MetaPhlAn accepts metagenomic sequence in several formats, including
`.fasta`, `.fastq`, `bowtie2out` and `sam`.

To see all the input formats (and other arguments), type `metaphlan -h | less`. Use the
arrow key to move up and down. Type `q` to quit back to the prompt.

    usage: metaphlan --input_type {fastq,fasta,bowtie2out,sam} [--force]
                 [--bowtie2db METAPHLAN_BOWTIE2_DB] [-x INDEX]
                 [--bt2_ps BowTie2 presets] [--bowtie2_exe BOWTIE2_EXE]
                 [--bowtie2_build BOWTIE2_BUILD] [--bowtie2out FILE_NAME]
                 [--min_mapq_val MIN_MAPQ_VAL] [--no_map] [--tmp_dir]
                 [--tax_lev TAXONOMIC_LEVEL] [--min_cu_len]
                 [--min_alignment_len] [--add_viruses] [--ignore_eukaryotes]
                 [--ignore_bacteria] [--ignore_archaea] [--stat_q]
                 [--perc_nonzero] [--ignore_markers IGNORE_MARKERS]
                 [--avoid_disqm] [--stat] [-t ANALYSIS TYPE]
                 [--nreads NUMBER_OF_READS] [--pres_th PRESENCE_THRESHOLD]
                 [--clade] [--min_ab] [-o output file] [--sample_id_key name]
                 [--use_group_representative] [--sample_id value]
                 [-s sam_output_file] [--legacy-output] [--CAMI_format_output]
                 [--unknown_estimation] [--biom biom_output] [--mdelim mdelim]
                 [--nproc N] [--install] [--force_download]
                 [--read_min_len READ_MIN_LEN] [-v] [-h]
                 [INPUT_FILE] [OUTPUT_FILE]

    DESCRIPTION
     MetaPhlAn version 3.0 (20 Mar 2020): 
     METAgenomic PHyLogenetic ANalysis for metagenomic taxonomic profiling.

    AUTHORS: Nicola Segata (nicola.segata@unitn.it), Duy Tin Truong, Francesco Asnicar (f.asnicar@unitn.it), 
    Francesco Beghini (francesco.beghini@unitn.it)

<!--[](https://github.com/biobakery/biobakery/blob/master/images/MetaPhlAn3_help.png)-->

------------------------------------------------------------------------

-   **Which command line arguments are required?**
-   **Which optional command line arguments seem most important or most
    commonly used?**

------------------------------------------------------------------------

For the purpose of this tutorial, we will use the following set of six
input files that have been subsampled for rapid analysis:

-   [SRS014476-Supragingival\_plaque.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014476-Supragingival_plaque.fasta.gz)
-   [SRS014494-Posterior\_fornix.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014494-Posterior_fornix.fasta.gz)
-   [SRS014459-Stool.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014459-Stool.fasta.gz)
-   [SRS014464-Anterior\_nares.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014464-Anterior_nares.fasta.gz)
-   [SRS014470-Tongue\_dorsum.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014470-Tongue_dorsum.fasta.gz)
-   [SRS014472-Buccal\_mucosa.fasta.gz](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/input/SRS014472-Buccal_mucosa.fasta.gz)

The original files, and many others, can be downloaded from the [HMP
DACC](http://hmpdacc.org/HMASM/).


Please proceed to [**Run a single sample**](#run-a-single-sample) section below. 

***

------------------------------------------------------------------------

-   **If you're familiar with the command line and file structure, does
    it matter where you place MetaPhlAn's input files, or where you run
    it from?**
-   **What about these input files might make them particularly
    appropriate for a short demonstration?**

------------------------------------------------------------------------

### **Run a single sample**

Here is the basic example to profile a single metagenome from raw reads:

```
metaphlan data/SRS014476-Supragingival_plaque.fasta --input_type fasta --nproc 4 --force -o output/SRS014476-Supragingival_plaque_profile.txt --bowtie2out output/SRS014476-Supragingival_plaque.fasta.bowtie2out.txt
```

### **Output files**

Running MetaPhlAn, following the example in the prior section, will
create two output files. Check what files have been created with `more`.

**File 1:**
[SRS014476-Supragingival\_plaque.fasta.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/output/SRS014476-Supragingival_plaque.fasta.gz.bowtie2out.txt)

This file contains the intermediate mapping results to unique sequence
markers.

Alignments are listed one per line in tab-separated columns of read and
reference marker.
```
more output/SRS014476-Supragingival_plaque.fasta.bowtie2out.txt
```
Output:

    HWUSI-EAS1568_102539179:1:100:10001:7882/1      712117__F3PCC2__HMPREF9056_02717
    HWUSI-EAS1568_102539179:1:100:10007:17628/1     712357__A0A0K2JD54__AMK43_09330
    HWUSI-EAS1568_102539179:1:100:10017:5224/1      2047__E3H3C1__HMPREF0733_12099
    HWUSI-EAS1568_102539179:1:100:10023:7402/1      712122__A0A0M4H4P0__AM609_02120
    HWUSI-EAS1568_102539179:1:100:10025:16605/1     43768__C0E1K0__murJ
    HWUSI-EAS1568_102539179:1:100:10033:18381/1     43768__E0DI09__HMPREF0299_5319
    HWUSI-EAS1568_102539179:1:100:10083:4412/1      2047__E3H2T8__coaBC
    HWUSI-EAS1568_102539179:1:100:10091:12482/1     505__C4GHX6__GCWU000324_00464
    HWUSI-EAS1568_102539179:1:100:10094:10442/1     544581__U1RE23__HMPREF1979_00736
    HWUSI-EAS1568_102539179:1:100:10103:1753/1      28133__F9DGE7__CBG57_05925
    HWUSI-EAS1568_102539179:1:100:10109:14464/1     43768__E0DGH3__HMPREF0299_6971
    HWUSI-EAS1568_102539179:1:100:10112:17904/1     43768__E0DEQ2__HMPREF0299_6337


**File 2:**
[SRS014476-Supragingival\_plaque\_profile.txt](https://github.com/biobakery/biobakery/raw/master/demos/biobakery_demos/data/metaphlan3/output/SRS014476-Supragingival_plaque_profile.txt)

This file contains the final computed organism abundances.

Organism abundances are listed one clade per line, tab-separated from
the clade's percent abundance:

     more output/SRS014476-Supragingival_plaque_profile.txt

Output:

    #mpa_v30_CHOCOPhlAn_201901
    #/n/huttenhower_lab/tools/metaphlan3/bin/metaphlan SRS014476-Supragingival_plaque.fasta --input_type fasta
    #SampleID       Metaphlan_analysis
    #clade_name     NCBI_tax_id     relative_abundance      additional_species
    k__Bacteria     2       100.0   
    k__Bacteria|p__Actinobacteria   2|201174        100.0   
    k__Bacteria|p__Actinobacteria|c__Actinobacteria 2|201174|1760   100.0   
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales    2|201174|1760|85007     65.25681        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales        2|201174|1760|85006     34.74319        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae      2|201174|1760|85007|1653        65.25681        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae      2|201174|1760|85006|1268        34.74319        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae|g__Corynebacterium   2|201174|1760|85007|1653|1716   65.25681        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae|g__Rothia    2|201174|1760|85006|1268|32207  34.74319        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae|g__Corynebacterium|s__Corynebacterium_matruchotii    2|201174|1760|85007|1653|1716|43768     65.25681        
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae|g__Rothia|s__Rothia_dentocariosa     2|201174|1760|85006|1268|32207|2047     34.74319        k__Bacteria|p__Actinobacteria|
    ~
    ~

* The file has a 4-line header. The **first line** lists the reference marker genes database that MetaPhlAn uses. There are ~1.1M unique clade-specific marker genes identified from ~100k reference genomes (~99,500 bacterial and archaeal and ~500 eukaryotic). The **second line** lists the path to the tool, the name of the input file and the arguments that mentioned. The **fourth line** has the column headers for the columns below.
* The first column lists clades, ranging from taxonomic kingdoms
(Bacteria, Archaea, etc.) through species. The taxonomic level of each
clade is prefixed to indicate its level:
`Kingdom: k__, Phylum: p__, Class: c__, Order: o__, Family: f__, Genus: g__, Species: s__`. Let us examine these more clearly by listing them out as per taxonomic hierarchy.

We will look for (grep) lines which contain the pattern `s__` that is associated with species and print the first match with the `-m1` argument. Remember that this file have 4 tab-separated columns and the taxonomy is listed in the first; so we will use `cut -f1` to get (cut out) the first column only (the field at position 1). Finally, the taxonomic levels are separated by the `|` character which we will replace with the new line character `\n`.


      grep "s__" -m1 output/SRS014476-Supragingival_plaque_profile.txt | cut -f1 | sed 's/|/\n/g'

Output (The taxonomy of the microbe *C. matruchotii*):

    k__Bacteria
    p__Actinobacteria
    c__Actinobacteria
    o__Corynebacteriales
    f__Corynebacteriaceae
    g__Corynebacterium
    s__Corynebacterium_matruchotii


* The second column lists the respective NCBI Taxon IDs.
* The third column lists relative abundances. Since sequence-based profiling is relative and does not provide absolute
cellular abundance measures, clades are hierarchically summed. Each
level will sum to 100%; that is, the sum of all kingdom-level clades is
100%, the sum of all genus-level clades (including unclassified) is also
100%, and so forth. OTU equivalents can be extracted by using only the
species-level `s__` clades from this file (again, making sure to include
clades unclassified at this level).

Let us check if all orders sum to 100% using `grep`. The orders 'Corynebacteriales' and 'Micrococcales' are in the class 'Actinobacteria'. 


      grep o__ output/SRS014476-Supragingival_plaque_profile.txt | grep -v f__

Output:

    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales	2|201174|1760|85007	65.25681	
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales	2|201174|1760|85006	34.74319	
    
Similarly, the families must sum to 100%. In this example, let us also display the fields of interest i.e. taxonomy names and percentages for ease of viewing.
  
     grep f__ output/SRS014476-Supragingival_plaque_profile.txt | grep -v g__ | cut -f1,3

Output:

    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae	65.25681
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae	34.74319



* The fourth column lists additional species for cases where the metagenome profile contains clades that represent multiple species. The species listed in column 1 is the representative species in such cases.

------------------------------------------------------------------------

-   **What do the parts of the read identifiers in the first column of
    the first, per-read/marker output file indicate?**
-   **What do the parts of the gene identifiers in the second column of
    the first, per-read/marker output file indicate?**
-   **Since the second, per-clade abundance output file is already
    normalized, you never need to sum-normalize these relative
    abundances. However, if you tried to, what would the sum of all
    clades' relative abundances be?**

------------------------------------------------------------------------

### **bowtie2out file as input**

If available, it is recommended to use the bowtie2out file as an input to MetaPhlAn as it significantly speeds up metagenomic profiling. Let us delete the **File 2** we created in the previous step and use the bowtie2out file (**File1**) to regenerate it. Notice that we will have to change the `--input_type` argument.

     rm -f output/SRS014476-Supragingival_plaque_profile.txt
     metaphlan output/SRS014476-Supragingival_plaque.fasta.bowtie2out.txt --input_type bowtie2out > output/SRS014476-Supragingival_plaque_profile.txt

On real data, you will notice that this executes much more quickly than on the raw FASTA file. This is especially helpful if all you want to do is change the format of the output or similar.

### **Run multiple samples**

- Each MetaPhlAn execution processes exactly one sample, but the
resulting single-sample analyses can easily be combined into an
abundance table spanning multiple samples. Let's put everything into a single script that we can run.
First, we will make a file that will contain the commands we want:
```
touch scripts/run_metaphlan.sh  #<<-- create an empty file
chmod +x scripts/run_metaphlan.sh  #<<-- make it executable
nano scripts/run_metaphlan.sh  #<<-- or your favorite text editor (I usually use vim, but the intro to linux on the first day used nano)
```
Then paste the following into this file:
```
#!/usr/bin/env bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

scriptFolder=`dirname $0`  #<<-- where this script is located
baseFolder=$(dirname $scriptFolder)  #<<-- the main analysis folder (one up from the script folder)
outputFolder="${baseFolder}/output"  #<<-- output folder

# Now analyze everything in one go
echo "Now running everything"  #<<-- print a message
inputFolder="${baseFolder}/data/"
for file in `ls ${inputFolder}/*.fasta`;
do
        baseName=$(basename $file)
        metaphlan $file --input_type fasta --nproc 1 --CAMI_format_output --force -o ${outputFolder}/${baseName%.fasta}.cami_profile --bowtie2out ${outputFolder}/${baseName}.bowtie2out.txt &
        metaphlan $file --input_type fasta --nproc 1 --force -o ${outputFolder}/${baseName%.fasta}.default_profile --bowtie2out ${outputFolder}/${baseName}.bowtie2out2.txt ;
done
```

At this point, you can then execute the script using the following command:
```
./scripts/run_metaphlan.sh
```
<!-- The `&` on the end means to send the process to the background, and the `nohup` asks the shell: "even though I don't have this process actively pulled up, please don't hang up on it". This will let the process run in the background as we continue the analysis below. -->

You will now have a complete set of six profile output files
and six intermediate mapping files. If you'd like to skip this step to
speed things up, the 12 demo file outputs can be downloaded from the
following links (right-click on the link and pick 'Save Link as ..' or click on the link and then right-click on the preview page and select "Save Page as...", or copy the URL to download on a server).

1. **Profile output files**

    -   [SRS014459-Stool\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014459-Stool_profile.txt)
    -   [SRS014464-Anterior\_nares\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014464-Anterior_nares_profile.txt)
    -   [SRS014470-Tongue\_dorsum\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014470-Tongue_dorsum_profile.txt)
    -   [SRS014472-Buccal\_mucosa\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014472-Buccal_mucosa_profile.txt)
    -   [SRS014476-Supragingival\_plaque\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014476-Supragingival_plaque_profile.txt)
    -   [SRS014494-Posterior\_fornix\_profile.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014494-Posterior_fornix_profile.txt)

2. **Intermediate mapping output files**

   -   [SRS014459-Stool.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014459-Stool.fasta.gz.bowtie2out.txt)
   -   [SRS014464-Anterior\_nares.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014464-Anterior_nares.fasta.gz.bowtie2out.txt)
   -   [SRS014470-Tongue\_dorsum.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014470-Tongue_dorsum.fasta.gz.bowtie2out.txt)
   -   [SRS014472-Buccal\_mucosa.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014472-Buccal_mucosa.fasta.gz.bowtie2out.txt)
    -   [SRS014476-Supragingival\_plaque.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014476-Supragingival_plaque.fasta.gz.bowtie2out.txt)
    -   [SRS014494-Posterior\_fornix.fasta.gz.bowtie2out.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/SRS014494-Posterior_fornix.fasta.gz.bowtie2out.txt)

## **Visualize results**
---------------------

### **Simple Vizualization with TAMPA**
For a quick visualization of the profile, I've developed a tool called TAMPA (TAxonoMic Profiling Anlaysis) to help view the profile output when it is in the CAMI 
(Critical Assessment of Metagenome Interpretation) format. 

If you are on OpenDemand, TAMPA comes pre-installed, and you can activate it with
```
conda deactivate
conda activate microbiome2
```

To install this tool from scratch, run the following:
```bash
cd ~
git clone https://github.com/dkoslicki/TAMPA.git
```

You can then create the visualization with the following command:
```bash
cd ~/MetaPhlAn_analysis
python ../TAMPA/src/profile_to_plot.py -i output/SRS014464-Anterior_nares.cami_profile -g output/SRS014464-Anterior_nares.cami_profile -b output/Anterior_nares -nm genus
```
This will create a file `Anterior_nares_tree_genus_Metaphlan_analysis.png` which you can transfer back to your device and view. It should look like the following:
![Anterior_nares_tree_genus_Metaphlan_analysis](https://user-images.githubusercontent.com/6362936/128067595-75f37852-9a16-4762-9e8f-529ed2f71980.png)

Note that TAMPA was originally designed for pairwise comparison of profiles (tool vs. tool, or tool vs. ground truth), hence the funky display.

### **Merge outputs for hclust**

Finally, the MetaPhlAn distribution includes a utility script that will
create a single tab-delimited table from these files: 
```bash
merge_metaphlan_tables.py output/*default_profile > output/merged_abundance_table.txt
```
-   [merged\_abundance\_table.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance_table.txt)

The resulting table can be opened in Excel, any gene expression analysis
program, `less` (example below), or visualized graphically as per
subsequent tutorial sections:

     cat output/merged_abundance_table.txt | column -t | more

The first few lines look like:

    #mpa_v30_CHOCOPhlAn_201901
    clade_name      NCBI_tax_id     SRS014494-Posterior_fornix_profile      SRS014476-Supragingival_plaque_profile  SRS014472-Buccal_mucosa_profile SRS014470-Tongue_dorsum_profile SRS014464-Anterior_nares_profile        SRS014459-Sto
    k__Bacteria     2       100.0   100.0   100.0   100.0   100.0   100.0
    k__Bacteria|p__Actinobacteria   2|201174        0       100.0   0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria 2|201174|1760   0       100.0   0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales    2|201174|1760|85007     0       65.25681        0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae      2|201174|1760|85007|1653        0       65.25681        0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae|g__Corynebacterium   2|201174|1760|85007|1653|1716   0       65.25681        0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Corynebacteriales|f__Corynebacteriaceae|g__Corynebacterium|s__Corynebacterium_matruchotii    2|201174|1760|85007|1653|1716|43768     0       65.25681        0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales        2|201174|1760|85006     0       34.743190000000006      0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae      2|201174|1760|85006|1268        0       34.743190000000006      0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae|g__Rothia    2|201174|1760|85006|1268|32207  0       34.743190000000006      0       0       0       0
    k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Micrococcales|f__Micrococcaceae|g__Rothia|s__Rothia_dentocariosa     2|201174|1760|85006|1268|32207|2047     0       34.743190000000006      0       0       0       0

<!--[](https://github.com/biobakery/biobakery/blob/master/images/MetaPhlAn3_tutorial_merged_abundance.png)-->

------------------------------------------------------------------------

-   **How might you convert these relative abundance measures to
    pseudo-RPKs that are sensitive to each sample's read depth?**
-   **In what important ways does analysis of a relative abundance table
    differ from that of a gene expression (microarray or RNA-seq)
    transcript table? In what ways are they similar?**
-   **Under what circumstances is this tab-delimited text data format
    particularly efficient or inefficient? Is this likely to be a
    problem for species-level taxonomic profiles?**

------------------------------------------------------------------------


### **Create a heatmap with hclust2**

A heatmap is one way to visualize tabular abundance results such as
those from MetaPhlAn. The plotting tool we'll use here, `hclust2`, is a
convenience script that can show any, some, or all of the microbes or
samples in a MetaPhlAn table. In this tutorial we will plot the heatmap
for all of the samples.

If you are on OpenDemand, hclust2 is already installed in the `microbiome2` environment:
```
conda deactivate
conda activate microbiome2
```

Otherwise, if you are on a personal machine or somewhere else, you can install hclust2 via conda:

     conda install -c biobakery hclust2


------------------------------------------------------------------------

**Step 1:** Generate the species only abundance table

Run the following command to create a species only abundance table,
providing the abundance table (
[merged\_abundance\_table.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance_table.txt)
) created in prior tutorial steps: :

```
  grep -E "s__|clade" output/merged_abundance_table.txt | sed 's/^.*s__//g'\
| cut -f1,3-8 | sed -e 's/clade_name/body_site/g' > output/merged_abundance_table_species.txt
```

There are four parts to this command. The first grep searches the file
for the regular expression `"s__|clade"` which matches to those lines
with species information and also to the header which contains the names of the body sites. The
`sed` removes the full taxonomy from each line so the first column only
includes the species name. The `cut` gives us all columns except the NCBI Taxon ID (column 2) and the last `sed`
helps us replace `clade_name` to `body_site`.

The new abundance table
([merged\_abundance\_table\_species.txt](https://github.com/biobakery/biobakery/blob/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance_table_species.txt))
will contain only the species abundances with just the species names
(instead of the full taxonomy).

Take a look at the file with:
```bash
cat output/merged_abundance_table_species.txt | column -t | more
```
The first few lines of the file will look like: 

    body_site	SRS014494-Posterior_fornix_profile	SRS014476-Supragingival_plaque_profile	SRS014472-Buccal_mucosa_profile	SRS014470-Tongue_dorsum_profile	SRS014464-Anterior_nares_profile	SRS014459-Stool_profile
    Corynebacterium_matruchotii	0	65.25681	0	0	0	0
    Rothia_dentocariosa	0	34.743190000000006	0	0	0	0
    Bacteroides_stercoris	0	0	0	0	0	31.62003
    Prevotella_histicola	0	0	0	51.36481	0	0
    Prevotella_pallens	0	0	0	5.10895	0	0
    Gemella_haemolysans	0	0	19.17145	0	0	0
    Dolosigranulum_pigrum	0	0	0	0	2.7635099999999997	0
    Lactobacillus_crispatus	80.56118000000001	0	0	0	0	0
    Lactobacillus_iners	19.43882	0	0	0	0	0
------------------------------------------------------------------------

-   **Why is it useful to visualize (and particularly to cluster) only
    the "tips" of the taxonomic tree?**

------------------------------------------------------------------------

**Step 2:** Generate the heatmap

Next generate the species only heatmap by running the following command:

```
 hclust2.py -i output/merged_abundance_table_species.txt -o output/abundance_heatmap_species.png --f_dist_f braycurtis --s_dist_f braycurtis --cell_aspect_ratio 0.5 -l --flabel_size 10 --slabel_size 10 --max_flabel_len 100 --max_slabel_len 100 --minv 0.1 --dpi 300
```

We have only 16 microbes in this demo file but typically, for ease of viewing, one can show the top 25 species using the `--ftop 25` argument. This script uses
Bray-Curtis as the distance measure both between samples (s) and between
features (f) (microbes), sets the ratio between the width/height of cells to
0.5, uses a log scale for assigning heatmap colors, sets the sample and
feature label size to 10, sets the max sample and feature label length to
100, selects the minimum value to display as 0.1, and selects an image
resolution of 300 (in that order!).

Open the resulting heatmap
([abundance\_heatmap\_species.png](https://github.com/biobakery/biobakery/blob/mpa_docker/images/MetaPhlAn3_tutorial_abundance_heatmap.png))
to take a look. If you generated it on your local computer, just double
click. If you're using a server with just a terminal interface, you
might have to transfer the file locally first using a tool like `scp`.
If you're using a server with a graphical interface, you can open the
file using a command like `see abundance_heatmap_species.png`). Using
any of these methods, the results should look like:

<img src = "https://github.com/biobakery/biobakery/blob/mpa_docker/images/MetaPhlAn3_tutorial_abundance_heatmap.png" width=450>

Notice that due to the very large differences between body site
communities in the human microbiome, we can still easily see
site-specific species despite the small demonstration input files (each
is subsampled to only 10,000 reads for efficiency).

------------------------------------------------------------------------

-   **Which microbes are most abundant at each body site in these
    demonstration data?**
-   **Under what circumstances is log-scaling the heatmap abundance
    colors good? When might it be bad (i.e. visually deceptive)?**

------------------------------------------------------------------------
### **Create a cladogram with GraPhlAn**
You can also visualize microbial abundances on a tree of life (also
referred to as a phylogeny or cladogram) that captures their taxonomic
(or phylogenetic) relatedness. Here, we'll use a tool called
[GraPhlAn](http://huttenhower.sph.harvard.edu/graphlan) that can render
trees and annotate them with microbial names or data such as abundances.
The instructions here assume that you will run GraPhlAn from the command
line, but if you'd like to use an online
[Galaxy](https://galaxyproject.org) module instead, see the section on
[GraPhlAn in
Galaxy](https://github.com/biobakery/biobakery/wiki/graphlan#rst-header-graphlan-galaxy-module).
For more information on this tool, refer to the [GraPhlAn
tutorial](https://github.com/biobakery/biobakery/wiki/graphlan).


If you are on OpenDemand, you can activate GraPhlAn with the following
```
conda deactivate
conda activate /gpfs/group/RISE/training/2022_microbiome/graphlan
```

Otherwise, if you are on a personal machine or the like, you can install GraPhlAn with
[Conda](https://docs.conda.io/en/latest/):

```bash
conda deactivate
conda create -y -n graphlan biopython graphlan export2graphlan
conda activate graphlan 
```

This will install GraPhlAn, export2graphlan, and all of its dependencies.

------------------------------------------------------------------------
**Step 1:** Create the GraPhlAn input files

GraPhlAn requires two inputs: (i) a tree structure to represent and (ii)
graphical annotation options for the tree.

Run the following command to generate the two input files for GraPhlAn
(the tree and annotation files) providing the abundance table
([merged\_abundance\_table.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance_table.txt)) created in the prior tutorial steps reformatted to remove the version header and the NCBI taxon id in the second column.

     tail -n +2 output/merged_abundance_table.txt | cut -f1,3- > output/merged_abundance_table_reformatted.txt
     export2graphlan.py --skip_rows 1 -i output/merged_abundance_table_reformatted.txt --tree output/merged_abundance.tree.txt --annotation output/merged_abundance.annot.txt --most_abundant 100 --abundance_threshold 1 --least_biomarkers 10 --annotations 5,6 --external_annotations 7 --min_clade_size 1


The command above has options to skip rows 1 and 2 (headers), select the top 100
most abundance clades to highlight, set a minimum abundance threshold
for clades to be annotated, extract a minimum of 10 biomarkers, select
taxonomic levels 5 and 6 to be annotated within the tree, select
taxonomic level 7 to be used in the external legend, and set the minimum
size of clades annotated as biomarkers to 1. The output files created
are `merged_abundance.tree.txt` and `merged_abundance.annot.txt`.

------------------------------------------------------------------------

-   **What are the contents and structure of the "tree" file?**
-   **What are the contents and structure of the "annot" file?**

------------------------------------------------------------------------
**Step 2:** Create a cladogram

Run the following commands to generate the cladogram providing the tree
(
[merged\_abundance.tree.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance.tree.txt)
) and its annotation (
[merged\_abundance.annot.txt](https://github.com/biobakery/biobakery/raw/mpa_docker/demos/biobakery_demos/data/metaphlan3/output/merged_abundance.annot.txt)
) files from the prior step :

```
graphlan_annotate.py --annot output/merged_abundance.annot.txt output/merged_abundance.tree.txt output/merged_abundance.xml
graphlan.py --dpi 300 output/merged_abundance.xml output/merged_abundance.png --external_legends
```

The first command creates an xml file from the tree and annotation
inputs. The second command creates the image, sets the image
resolution to 300 and requests external legends.

The first few lines of the xml file are: 

    <phyloxml xmlns="http://www.phyloxml.org" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.phyloxml.org http://www.phyloxml.org/1.10/phyloxml.xsd">
      <phylogeny rooted="true">
        <clade>
          <clade>
            <name>k__Bacteria</name>
            <branch_length>1.0</branch_length>
            <property applies_to="clade" datatype="xsd:string" id_ref="clade_marker_size" ref="A:1">10.0</property>
            <clade>

The generated cladogram
([merged\_abundance.png](https://github.com/biobakery/biobakery/blob/mpa_docker/images/MetaPhlAn3_tutorial_GraPhlAn_main.png))
is:

<img src = "https://github.com/biobakery/biobakery/blob/mpa_docker/images/MetaPhlAn3_tutorial_GraPhlAn_main.png"
width=500>


The annotation
([merged\_abundance\_annot.png](https://github.com/biobakery/biobakery/blob/mpa_docker/images/MetaPhlAn3_tutorial_GraPhlAn_annot.png)
) should be:

<img src = "https://github.com/biobakery/biobakery/blob/master/images/MetaPhlAn3_tutorial_GraPhlAn_annot.png"
width=450>


And the legend
([merged\_abundance\_legend.png](https://github.com/biobakery/biobakery/blob/master/images/MetaPhlAn3_tutorial_GraPhlAn_legend.png)
) is:

<img src = "https://github.com/biobakery/biobakery/blob/master/images/MetaPhlAn3_tutorial_GraPhlAn_legend.png"
width=350>


As above, if you generated these images on your local computer, open
them by simply double clicking. If you're using a server with only a
terminal interface, transfer the file locally first using a tool like
`scp`. If you're using a server with a graphical interface, you can open
the file using a command like `see merged_abundance.png`).

------------------------------------------------------------------------

-   **What is the PhyloXML format? Why might it be used in this
    context?**
-   **Why is it often particularly useful to plot circular, rather than
    linear, cladograms?**
-   **What other types of annotations might be useful on such a tree
    (either different graphical formats, or different types of data to
    take advantage of them)?**

------------------------------------------------------------------------

# Please proceed to the [mOTUs2 Section](TaxonomicProfiling-mOTUs2.md)
