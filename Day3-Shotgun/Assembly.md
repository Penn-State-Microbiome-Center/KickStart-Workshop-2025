# Metagenomic Assembly
(a.k.a. and you thought isolate genome assembly was hard)

Metagenomic assembly is one of the most computationally intensive part of WGS metagenomic analysis. This is due in part to how tangled the assembly graphs can look. For an 
illustration, here is a de Bruijn graph with k-mer size 50 for a mock metagenome with only 113 microorganisms inside of it:
![simLC fasta+simHC fasta-N12-smaller-l-50](https://user-images.githubusercontent.com/6362936/128256031-fb788323-583b-41b8-b02c-8c0a2ed86d74.PNG)
Disentangling that big knot into linear contigs requires quite a bit of clever algorithmic approaches, and is one reason why metagenome assembly is still a field of active development.

One of the most successful current approaches is to construct assembly/de Bruijn graphs with multiple k-mer sizes. We will be looking at a couple of such tools that 
were top performers in the recent [CAMI2 competition](https://www.biorxiv.org/content/10.1101/2021.07.12.451567v1).

1. [MEGAHIT](MEGAHIT.md)
2. [GATB](GATB.md)


Since we do not want to wait hours/days to complete assemblies, we will be using extremely small data sets. This might give you the impression that the tools are not 
resource intensive, but do not be decieved! The amount of resources required does not scale linearly with the number of reads (it grows much faster than that).

# Please proceed to the [MEGAHIT](MEGAHIT.md) section
