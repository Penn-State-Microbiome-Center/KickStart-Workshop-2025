# Genome binning with MetaBinner

MetaBinner is an ensemble-based approach to genome binning, originating in 2019. The basic workflow can be found in the [associated preprint](https://www.biorxiv.org/content/10.1101/2021.07.25.453671v1):
![MetaBinner](https://user-images.githubusercontent.com/6362936/128402144-07f5135e-d36f-4cc7-b2eb-a6e6f1019919.PNG)

# Installation and setup

## File directory setup and data acquisition
We will use the usual file structure and copy over the data from both the GATB and MEGAHIT analysis:
```
mkdir MetaBinner_analysis
cd MetaBinner_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt  #<<--TODO: change to single file if needed
ls *.gz | xargs -P6 -I{} gunzip {}
cp ../../MEGAHIT_analysis/output/default/final.contigs.fa MEGAHIT_default_contigs.fasta
cp ../../GATB_analysis/output/default.fasta GATB_default_contigs.fasta
cd ..
```

## Installing MetaBinner
MetaBinner can currently be installed via the following:
```bash
conda deactivate
cd scripts
git clone https://github.com/ziyewang/MetaBinner.git
cd MetaBinner
conda env create -y -f metabinner_env.yaml
conda activate metabinner_env  #<<-- note that we did not give this name to the environment; the name is contained in the yaml file itself.
conda install -y -c bioconda prodigal hmmer pplacer pandas
cd CheckM-1.0.18
python setup.py install
mkdir checkm_data
cd checkm_data
wget https://data.ace.uq.edu.au/public/CheckM_databases/checkm_data_2015_01_16.tar.gz
tar -xzf checkm_data_2015_01_16.tar.gz 
checkm data setRoot .
cd ../../../..
```
Please note that this is a Linux-specific installation, but the developers are [working](https://github.com/ziyewang/MetaBinner/issues/4) on a cross-platform installation method (i.e. conda).

## Running MetaBinner

### Preprocessing
MetaBinner requires a bit more manual work as we will need to create the coverage profile and k-mer spectrum before running MetaBinner.

1. Filter out short contigs

MetaBinner provides a script to do this, but here's a much faster way to do it. _Note_ if you were doing this on real data, you would want to replace the `250` with something like `1000`.
```
awk '!/^>/{next}{getline s} length(s) >= 250 { print $0 "\n" s}' data/MEGAHIT_default_contigs.fasta > data/MEGAHIT_default_contigs_longer.fasta
awk '!/^>/{next}{getline s} length(s) >= 250 { print $0 "\n" s}' data/GATB_default_contigs.fasta > data/GATB_default_contigs_longer.fasta
```

2. Generate coverage profiles

This will map the reads back to the assembled contigs to get an idea of the read coverage per contig. Let's do this for both the MEGAHIT and GATB assemblies:
```
bash scripts/MetaBinner/scripts/gen_coverage_file.sh -a data/MEGAHIT_default_contigs_longer.fasta -o output/on_MEGAHIT -t 4 -m 40 --single-end data/SRS014464-Anterior_nares.fastq
bash scripts/MetaBinner/scripts/gen_coverage_file.sh -a data/GATB_default_contigs_longer.fasta -o output/on_GATB -t 4 -m 40 --single-end data/SRS014464-Anterior_nares.fastq
```

3. Generate 4-mer frequency spectrum

Basically, we need to calculate the frequency of every k-mer for `k=4` in each of the contigs. There is an option to ignore shorter contigs, so we will set this really low 
since we are dealing with such a small test dataset. You can also specify the k-mer size, but _please_ do not try it for any `k>4`, the implementation is a brute force enumeration of k-mers, and [much](https://gatb.inria.fr/software/dsk/), [more](http://www.genome.umd.edu/jellyfish.html), [efficient](https://khmer.readthedocs.io/en/latest/), [methods](https://github.com/refresh-bio/KMC), [exist](https://github.com/uni-halle/gerbil), [for](https://github.com/pmelsted/BFCounter), [larger](https://sourceforge.net/projects/kanalyze/), [k-sizes](http://grafia.cs.ucsb.edu/msp/download.html).
We will make the 4-mer frequency spectrums for both assemblies:
```bash
python scripts/MetaBinner/scripts/gen_kmer.py data/MEGAHIT_default_contigs_longer.fasta 250 4; mv data/kmer_4_f250.csv output/on_MEGAHIT/kmer_4_f250.csv
python scripts/MetaBinner/scripts/gen_kmer.py data/GATB_default_contigs_longer.fasta 250 4; mv data/kmer_4_f250.csv output/on_GATB/kmer_4_f250.csv
```

### Running the tool

```
#!/bin/bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

baseDir=/home/dmk333/KickStartWorkshop2021/MetaBinner_analysis/
# Full MetaBinner Path
metabinnerPath=${baseDir}/scripts/MetaBinner

#path to the input files for MetaBinner and the output dir:
contigFile=${baseDir}/data/MEGAHIT_default_contigs_longer.fasta
outputDir=${baseDir}/output/on_MEGAHIT
coverageProfiles=${baseDir}/output/on_MEGAHIT/coverage_profile.tsv
kmerProfile=${baseDir}/output/on_MEGAHIT/kmer_4_f250.csv


bash ${metabinnerPath}/run_metabinner.sh -a ${contigFile} -o ${outputDir} -d ${coverageProfiles} -k ${kmerProfile} -p ${metabinnerPath}
```

```
2021-08-05 16:25:09,769 - Contig_file:  /home/dmk333/KickStartWorkshop2021/MetaBinner_analysis//data/MEGAHIT_default_contigs_longer.fasta
2021-08-05 16:25:09,769 - Coverage_profiles:    /home/dmk333/KickStartWorkshop2021/MetaBinner_analysis//output/on_MEGAHIT/coverage_profile.tsv
2021-08-05 16:25:09,769 - Composition_profiles: /home/dmk333/KickStartWorkshop2021/MetaBinner_analysis//output/on_MEGAHIT/kmer_4_f250.csv
2021-08-05 16:25:09,769 - Output file path:     /home/dmk333/KickStartWorkshop2021/MetaBinner_analysis//output/on_MEGAHIT/metabinner_res/result.tsv
```
