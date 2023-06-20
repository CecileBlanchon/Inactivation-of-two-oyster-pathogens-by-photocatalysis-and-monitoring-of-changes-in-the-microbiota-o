# ASV_Script.md

This script present the step performed to analyse the sequencing data

**20220905_DataPhotomic2**  # folder with all forward and reverse sequences per sample \
**MetadatFile3_ASV.txt**  # metadata file in tsv format for Photomic 2 data (separated by tabs) 

--------------------------------------------------------------------------------------------------------------------------------
For this script the mac terminal was used in bash.\
All the .qzv files generated during this script can be visualized with the *qiime tools view* command or on https://view.qiime2.org/

## Installation of Conda and Qiime ##
  ### Download of Miniconda  ###
> bash-3.2$ curl -O https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-MacOSX-arm64.pkg \
> bash-3.2$    conda init bash # for properly activated conda, the terminal need to be reboot after this step
  ### Download of Bioconda ###
> bash-3.2$ conda install -c bioconda bioconda-utils \
> bash-3.2$ conda config --add channels default \
> bash-3.2$ conda config --add channels bioconda # if warning bioconda are already install onto the computer \
> bash-3.2$ conda config --add channels conda-forge \
> bash-3.2$ conda install wget # pour installer wget 
  ### Download of Qiime 2 ###
> bash-3.2$ curl -sL "https://data.qiime2.org/distro/core/qiime2-2022.8-py36-osx-conda.yml" > "qiime2022.8.yml" 

**Creating a new working environment with conda** \
!!! **Conda working only with mac on 64 bits environment (osx-64 for intel Chip)** so if a the configuration is in M1 (apple M1 chip) **"CONDA_SUBDIR=osx-64" need to be add before the command**, the Rosetta terminal can also be used 

> bash-3.2$ CONDA_SUBDIR=osx-64  conda env create \
>             -n qiime2-2022.8 \
>             --file qiime2-2022.8.yml \
> bash-3.2$ rm qiime2-2022.8.yml # deleted of qiime2 \
> bash-3.2$ conda activate qiime2-2022.8 \
> bash-3.2$ source .bashrc # a shortcut has been created to solve the problem when using the conda activate command: $ echo 'source /opt/miniconda3/etc/profile.d/conda.sh' >> .bashrc \
> bash-3.2$ conda env config vars set CONDA_SUBDIR=osx-64 # change the conda env config so that it always works in CONDA_SUBDIR=osx-64 \
> bash-3.2$ conda deactivate \
> bash-3.2$ conda activate qiime2-2022.8 \
> (qiime2-2022.8) bash-3.2$ echo "CONDA_SUBDIR: $CONDA_SUBDIR" 

## importing sequencing data ##
**QIIME2 used artefact on the .qza format**, the sequencing data must be imported in qza format using the *qiime tools import* command. The sequencing data were obtained by **Illumina MiSeq from the V3-V4 fu region of the 16S rRNA gene**. These are **Paired End data already demiltiplexed** (*--input-format CasavaOneEightSingleLanePerSampleDirFmt*) in the form of **fastq** R1 and R2 files for forward and reverse sequencing of each sample (*--type 'SampleData[PairedEndSequencesWithQuality]'*), all of which are grouped together in the folder **"20220905_DataPhotomic2"**.

> (qiime2-2022.8) bash-3.2$ tar -xf 20220905_DataPhotomic2.tar.gz \
>                -C 20220905_DataPhotomic2 \
> (qiime2-2022.8) bash-3.2$ cd ./20220905_DataPhotomic2 \
> (qiime2-2022.8) bash-3.2$ gzip *fastq \
> (qiime2-2022.8) bash-3.2$ for f in *.fastq.gz; do gzip -tv $f; done # check files for damages \
> (qiime2-2022.8) bash-3.2$ cd /Users/cecileblanchon/Documents/ThèseUPVD/Résultats/20220622_PhotocatalyseVirus-Barcoding_qPCR/Barcoding/Test_ASV \
> (qiime2-2022.8) bash-3.2$ qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' \
>               --input-path 20220905_DataPhotomic2 \
>               --input-format CasavaOneEightSingleLanePerSampleDirFmt \
>               --output-path Photomic2.qza \
> (qiime2-2022.8) bash-3.2$ qiime tools peek Photomic2.qza # check everything is good \
> (qiime2-2022.8) bash-3.2$ qiime demux summarize --i-data Photomic2.qza \
>               --o-visualization Photomic2.qzv \
> (qiime2-2022.8) bash-3.2$ qiime tools view Photomic2.qzv 

## Denoising avec DADA2 ## (étape relativement longue **~5h**)
DADA2 is a pipeline for detecting and correcting Illumina amplicon sequences. Here you need to set the parameters *--p-trim-left* and *--p-trunc-len* to remove regions of poor quality.
In this step, DADA2 filters, merges and eliminates chimeras in the samples (see the following github for details: https://benjjneb.github.io/dada2/bigdata.html). \
Sequence filtration consists in eliminating low-quality zones. To increase the number of sequences retained, you need to play with the *--p-trunc* parameters, but be careful not to reduce them too much to maintain sufficient overlap to merge the sequences together. It's also important to eliminate the primers (*--p-trim*).

**Chimera elimination is carried out using the de novo method**, i.e. without a reference database, but from the data itself.

**Eliminating adapters is an important step, since we've used degenerate adapters with N (and therefore different lengths).**

> (qiime2-2022.8) bash-3.2$ qiime dada2 denoise-paired --i-demultiplexed-seqs Photomic2.qza \
>               --p-trim-left-f 15 \
>               --p-trim-left-r 21 \
>               --p-trunc-len-f 251 \
>               --p-trunc-len-r 230 \
>               --o-table table_PseudoPooling.qza \
>               --o-representative-sequences re-seqs_PseudoPooling.qza \
>               --o-denoising-stats denoising-stats-qza_PseudoPooling \
>               --p-pooling-method pseudo \
>               --p-chimera-method pooled \
> (qiime2-2022.8) bash-3.2$ qiime metadata tabulate   \
>               --m-input-file denoising-stats-qza_PseudoPooling.qza \  
>               --o-visualization stats-dada2.qzv \
> (qiime2-2022.8) bash-3.2$ qiime tools view stats-dada2.qzv  # **ASV creation statistics: number of sequences after each step (filtered, denoised, merged, non-chemical) --> between 10 and 62% of remaining sequences per sample** 

The choice of nucleotides for truncation is based on the quality (**qiime tools view Photomic2.qzv**) and size of the primers used.

Once you've cleaned the seqeunces and created the ASVs with DADA2, you can view the characteristics and statistics associated with this step.

> (qiime2-2022.8) bash-3.2$ qiime feature-table summarize \
>                        --i-table table_PseudoPooling.qza \
>                        --o-visualization table_PseudoPooling.qzv \
>  (qiime2-2022.8) bash-3.2$ qiime tools view table_PseudoPooling.qzv # **number of associated sequences per sample** \
> (qiime2-2022.8) bash-3.2$ qiime feature-table tabulate-seqs \
>                       --i-data re-seqs_PseudoPooling.qza \
>                        --o-visualization re-seqs_PseudoPooling.qzv \
> (qiime2-2022.8) bash-3.2$ qiime tools view re-seqs_PseudoPooling.qzv # **statistics on sequence lengths and detection frequencies** \
> (qiime2-2022.8) bash-3.2$ less dna-sequences.fasta \
> (qiime2-2022.8) bash-3.2$ grep -c ">" dna-sequences.fasta # 9 757 séquence dans re-seqs_PseudoPooling.qza \
> (qiime2-2022.8) bash-3.2$ qiime tools export --input-path denoising-stats-qza_PseudoPooling.qza --output-path ./ # creation of a tsv file with the **results contained in the file denoising-stats-qza_PseudoPooling.qza**, and also visible with the file **stats-dada2.qzv**.\
> (qiime2-2022.8) bash-3.2$ less feature-table.biom \
> (qiime2-2022.8) bash-3.2$ biom convert -i feature-table.biom -o feature-table_PseudoPooling.tsv --to-tsv # creation of a tsv file with the **results contained in the biom file feature-table.biom** = number of times seqeunce is found per samples

### Création d'un arbre phylogénétique pour les analyses de diversitées
The pipeline uses **FastTree (with the mafft program)** to perform multiple sequence alignments and generate a tree. 

> (qiime2-2022.8) bash-3.2$ qiime phylogeny align-to-tree-mafft-fasttree \
>                       --i-sequences re-seqs_PseudoPooling.qza \
>                       --o-alignment aligned-re-seqs_PseudoPooling2.qza \
>                       --o-masked-alignment masked-aligned-re-seqs_PseudoPooling2.qza \
>                       --o-tree unrooted-tree_PseudoPooling2.qza \
>                       --o-rooted-tree rooted-tree_PseudoPooling2.qza

### Rarefaction curves

> (qiime2-2022.8) bash-3.2$ qiime diversity alpha-rarefaction \
>                     --i-table table_PseudoPooling2.qza \
>                     --i-phylogeny rooted-tree_PseudoPooling2.qza \
>                     --p-max-depth 22000 \
>                     --m-metadata-file MetadataFile3_ASV.txt \
>                     --o-visualization alpha-rarefaction_PseudoPooling2.qzv \
> (qiime2-2022.8) bash-3.2$ qiime tools view alpha-rarefaction_PseudoPooling2.qzv 

2 graphs are obtained, the first representing **the alpha rarefaction curve**. All samples have reached the plateau, i.e. the entire diversity of echnatillons has been sequenced. The second graph shows the **number of snails remaining when the snails are grouped by group (according to the Metadata file) as a function of the sampling depth**.

### Assignation taxonomique
#### Training V3-V4 de la base de donnée Silva 138
Training is carried out on the Silva 138 database using the adapter sequences used for sequencing, i.e.: \
- forward: CCTACGGGNGGCWGCAG \
- reverse: GACTACHVGGGTATCTAATCC \
Silva data is retrieved and saved in the *TrainingSilva138*: \
- for the 16S sequence on https://data.qiime2.org/2022.8/common/silva-138-99-seqs.qza \
- for the corresponding taxonomic affiliation on https://data.qiime2.org/2022.8/common/silva-138-99-tax.qza \

> (qiime2-2022.8) bash-3.2$ cd /Users/cecileblanchon/Documents/ThèseUPVD/Résultats/20220622_PhotocatalyseVirus-Barcoding_qPCR/Barcoding/Test_ASV/TrainingSilva138 \
> (qiime2-2022.8) bash-3.2$ qiime feature-classifier extract-reads \
>                   --i-sequences silva-138-99-seqs.qza \
>                   --p-f-primer CCTACGGGNGGCWGCAG \
>                   --p-r-primer GACTACHVGGGTATCTAATCC \
>                   --p-min-length 100 \
>                   --p-max-length 600 \
>                   --o-reads ref-seqs_trainSilva2.qza \
> (qiime2-2022.8) bash-3.2$ qiime feature-classifier fit-classifier-naive-bayes \
>                   --i-reference-reads ref-seqs_trainSilva2.qza \
>                   --i-reference-taxonomy silva-138-99-tax.qza \
>                   --o-classifier Silva_V3V4_classifier2.qza
>

#### Assignation
**!!!** The scikit-learn version used to generate the classifier must be the same as the Qiime2 version installed, in my case **Qiime2 version 2022.8 (scikit-learn (0.24.1)**.

> (qiime2-2022.8) bash-3.2$ qiime feature-classifier classify-sklearn \
>                     --i-classifier Silva_V3V4_classifier2.qza \
>                     --i-reads re-seqs_PseudoPooling.qza \
>                     --o-classification taxonomy_SilvaTraining_Pooling2.qza \
> (qiime2-2022.8) bash-3.2$ qiime metadata tabulate \
>                     --m-input-file taxonomy_SilvaTraining_Pooling2.qza \
>                     --o-visualization taxonomy_SilvaTraining_Pooling2.qzv \
> (qiime2-2022.8) bash-3.2$ qiime taxa barplot \
>                     --i-table table.qza \
>                     --i-taxonomy taxonomy_SilvaTraining_Pooling2.qza \
>                     --o-visualization taxa-bar-plots-Silva_V3V4.qzv \
>                     --m-metadata-file MetadataFile3_ASV.txt \
> (qiime2-2022.8) bash-3.2$ qiime tools view taxa-bar-plots-Silva.qzv


## Création d'un fichier biom avec la taxonomy

> (qiime2-2022.8) bash-3.2$ qiime taxa collapse \
>                   --i-table table_PseudoPooling.qza \
>                   --i-taxonomy taxonomy_SilvaTraining_Pooling2.qza \
>                   --p-level 7 \
>                   --o-collapsed-table phyla-table.qza\
> (qiime2-2022.8) bash-3.2$ qiime tools export \
>                 --input-path phyla-table.qza \
>                 --output-path ./ # Exported phyla-table.qza as BIOMV210DirFmt to directory ./ 

