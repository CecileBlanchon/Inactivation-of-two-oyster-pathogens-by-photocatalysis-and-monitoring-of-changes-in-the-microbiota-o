# Article: Inactivation of two oyster pathogens by photo-oxidation in seawater: a case study of the effect of treatment on OsHV-1 µVar and *Vibrio harveyi*.

Cécile Blanchon<sup>1,2,3,4</sup>, Eve Toulza<sup>1</sup>, Christophe Calvayrac<sup>2,3</sup>, Stanislawa Eichendorff<sup>4</sup>, Marie-Agnès Travers<sup>1</sup>, Jeremie Vidal-Dupiol<sup>1</sup>, Caroline Montagnani<sup>1</sup>, Jean-Michel Escoubas<sup>1</sup>, Christophe Stavrakakis<sup>5</sup>, Gael Plantard<sup>4</sup>

1 IHPE, Université de Montpellier, CNRS, Ifremer, Université de Perpignan Via Domitia, Perpignan, France \
2 Biocapteurs Analyses Environnement, Université de Perpignan Via Domitia, 66000 Perpignan, France \
3 Laboratoire de Biodiversité et Biotechnologies Microbiennes (LBBM), Sorbonne Universités, CNRS, 66650 Banyuls sur Mer, France \
4 PROMES-CNRS UPR 8521, Process Material and Solar Energy, Rambla de la Thermodynamique, 66100 Perpignan, France \
5 Ifremer – Unité EMMA Expérimentale Mollusques Marins Atlantique, F-85230 Bouin, France 

This repository contain all de file used for the barcoding analysis present in our article.

## 1. Presentation of the data ##
The data present here (**20220905_DataPhotomic2**) are 16S rRNA gene of bacterial communities amplified usinf the variable V3-V4 loops (341F / 805R). This data were obtained by Paired-end sequencing performed on the MiSeq system (Illumina) at the Bio-Environment platform (UPVD). The **MetadataFile3_ASV.txt** present all the metadata needed for the analysis usinf the tsv format (tab used as separator).

## 2. The Amplicon Sequence Variants (ASV) creation ##
ASVs were created using bash language on a mac terminal. The used script (**ASV_Script.md**) present all the step we perform to obtain our results. During the ASV creation pipeline we also created the Phylogenetic tree used for the diversity analysis (**rooted-tree_PseudoPooling.qza**) and the Taxonomic affiliation (**taxonomy_SilvaTraining_Pooling2.qza**).

## 3. The statistical analysis ##
The statistical analysis were performed on R using a phyloseq object. The used script (**Photomic2-AnalyseEau-Article1.Rmd**) present all the used packages and the commands lines performed for the statistical analysis of our data.
