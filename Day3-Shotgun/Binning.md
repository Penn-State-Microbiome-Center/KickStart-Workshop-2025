# Metagenomic Binning

Metagenomic binning is the process of taking contigs and placing them in "bins." These bins are then labeled with taxa (if so, this is called _taxonomic binning_) or else 
labeled as individual genomes (if so, this is called _genome binning_). In short, this allows you to take your jumble of contigs from an assembly and partition them into either 
since genome bins, or bins belonging to individual taxa.

Importantly, binning is not the same as "read classification:" the process of (attempting to) assign taxa to individual reads or partition individual reads into single-genome origin 
bins. Currently, read classification is highly inaccurate for all data except that from long-read sequencing technologies. This has led to a fair bit of confusion though, 
and we will explore the impact this can have on your analysis by using a contig binner Kraken _incorrectly_ by using it on short reads rather than on contigs.

The following figure describes the current state of analysis for short read sequences:
![Slide2](https://user-images.githubusercontent.com/6362936/128400238-f9afa999-9a38-43a3-9412-813aa97908bd.PNG)


## Genome Binning
We will look at the genome binner [CONCOCT](https://concoct.readthedocs.io/en/latest/) in the [Genome Binning](GenomeBinning-CONCOCT.md) section.

## Taxonomic Binning
We will look at the taxonomic binner [Kraken2](https://ccb.jhu.edu/software/kraken2/index.shtml) in the [Taxonomic Binning](TaxonomicBinning-Kraken2.md) section.

# Please proceed to the [Genome Binning](GenomeBinning-CONCOCT.md) section
