# Day 2: Symbiosis and Amplicon analysis in R

## Morning Lecture:
Studying Symbiosis through Sequencing (Liana Burghardt) - [pdf](MicrobiomeKickStart_Burghardt_Aug2024.pdf)

## Amplicon Analysis in R: 
### Resources
- [2025 presentations PDF files](https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/evk5387_psu_edu/EvUN_epjzhtOs8fixwmt7vwB1vqGgXjR2VIcBbQCfU90Qw?e=PR420C) (need PSU authentication to access) 

#### **Suggested Reading**

Weinroth MD, Belk AD, Dean C, et al. Considerations and best practices in animal science 16S ribosomal RNA gene sequencing microbiome studies. J Anim Sci. 2022;100(2):skab346. doi:10.1093/jas/skab346 [Link](https://academic.oup.com/jas/article/100/2/skab346/6519592)

Ginnan N, Bordenstein SR. It is time to authenticate the Microbiome Sciences with accredited educational programs and departments. PLoS Biol. 2023 Dec 7;21(12):e3002420. doi: 10.1371/journal.pbio.3002420. PMID: 38060452; PMCID: PMC10703218. [Link](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.3002420)

Mirzayi C, Renson A; Genomic Standards Consortium; Reporting guidelines for human microbiome research: the STORMS checklist Nat Med. 2021 Nov;27(11):1885-1892. doi: 10.1038/s41591-021-01552-x. Epub 2021 Nov 17. PMID: 34789871; PMCID: PMC9105086.[Link](https://www.nature.com/articles/s41591-021-01552-x)

Callahan BJ, McMurdie PJ, Rosen MJ, Han AW, Johnson AJ, Holmes SP. DADA2: High-resolution sample inference from Illumina amplicon data. Nat Methods. 2016;13(7):581-583. doi:10.1038/nmeth.3869 [Link](https://www.nature.com/articles/nmeth.3869)

Gloor GB, Macklaim JM, Pawlowsky-Glahn V, Egozcue JJ. Microbiome Datasets Are Compositional: And This Is Not Optional. Front Microbiol. 2017;8:2224. Published 2017 Nov 15. doi:10.3389/fmicb.2017.02224 [Link](https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2017.02224/full)

# **Best practices for differential abundance analysis**

Article: [Beyond Normalization: Incorporating Scale Uncertainty in Microbiome and Gene Expression Analysis](https://www.biorxiv.org/content/10.1101/2024.04.01.587602v1.full)

Package: [ALDEx2](https://www.bioconductor.org/packages/release/bioc/html/ALDEx2.html)


#### **Required Software**

Software:
[R and Rstudio](https://cran.rstudio.com/)

Packages: 
[dada2](https://bioconductor.org/packages/release/bioc/html/dada2.html) , [phyloseq](https://bioconductor.org/packages/release/bioc/html/phyloseq.html), and [tidyverse](https://tidyverse.tidyverse.org/)

🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹🔹
# 🔹Extra Material🔹

## **Textbook on experimental design, bias, etc - Veterinary Epidemiologic Research**

[Veterinary Epidemiologic Research](https://projects.upei.ca/ver/)

Dohoo, I., Martin, W., & Stryhn, H. (2009). Veterinary epidemiologic research (2nd ed.). VER Inc

Veterinary Epidemiologic Research is a comprehensive text covering the key principles and methods used in veterinary epidemiologic research. It is written primarily for researchers and graduate students in veterinary epidemiology, but the material is equally applicable to those in related disciplines (human epidemiology, public health etc).

## **Online "textbook" on fundamental concepts of microbiome**
[Meet the Metaorganism](https://metaorganism.app/en)

#### **Short lectures**
[Link to an overview of sequencing methodologies](https://psu.mediaspace.kaltura.com/media/Next+Generation+Sequencing/1_12vugl0d)

[Link to an overview of bioinformatics](https://psu.mediaspace.kaltura.com/media/Overview+of+Bioinformatics+and+Diversity+Metrics/1_1ih13k9f)

## **Online educational game on fundamental concepts of microbiome**
[Tiny Biome Tales](https://microbiome.gamelabgraz.at/)
Article: [‘Tiny biome tales’: playing a game to understand the human microbiome](https://www.the-microbiologist.com/news/tiny-biome-tales-playing-a-game-to-understand-the-human-microbiome/3858.article)

## **Relevant TED talks**
Laura Boykin: [How we're using DNA tech to help farmers fight crop diseases](https://www.ted.com/talks/laura_boykin_how_we_re_using_dna_tech_to_help_farmers_fight_crop_diseases?utm_campaign=tedspread&utm_medium=referral&utm_source=tedcomshare)
Steffanie Strathdee: [How Sewage Saved My Husband's Life from a Superbug](https://youtu.be/AbAZU8FqzX4?si=JDqz6OFD1Vb9Y0Wu) and [How does a virus eat bacteria?](https://www.cnn.com/videos/tv/2022/07/08/steffanie-strathdee-life-itself-wellness.cnn)
Rob Knight: [How our microbes make us who we are](https://www.ted.com/talks/rob_knight_how_our_microbes_make_us_who_we_are?utm_campaign=tedspread&utm_medium=referral&utm_source=tedcomshare)

#### **Suggested Resources**

### Background: Coursera - gut check
It is great to start grappling with concepts - videos are short<br>
https://www.coursera.org/lecture/microbiome/welcome-to-the-course-AuMZk

### Coding:

#### 🧑‍💻 The best way to get started with bioinformatics
For getting used to best practices in coding in the command line<br>
https://www.biostarhandbook.com/index.html

##### 🖥️ Getting started with R
1.	https://www.codecademy.com/learn/learn-r
2.	https://www.codecademy.com/search?query=R
   
##### 💻 Getting started with BASH
Before you do biostars handbook do this quick shell one
https://swcarpentry.github.io/shell-novice/

##### ✅ Scientific rigor, reproducibility, version control, and code-alongs
Riffomonas has great tutorials to get started with R and reproducible research with version control is also a must.
https://riffomonas.org/

##### 🧬 16S analysis
**16S analysis in R**<br>
https://benjjneb.github.io/dada2/tutorial.html<br>

**16S analysis in Qiime2**<br>
https://docs.qiime2.org/2021.11/tutorials/overview/
