# Metabolic Modeling: Gapseq Workflow
##### Updated 12/12/2022 by Laura Schaerer

# Installation: 

## Cloning the development version of gapseq
git clone https://github.com/jotech/gapseq
cd gapseq

## Create and activate a conda environment "gapseq-dev"
conda env create -n gapseq-dev --file gapseq_env.yml
conda activate gapseq-dev

## install one additional R-package
R -e 'install.packages("CHNOSZ", dependencies = TRUE, repos = "https://cloud.r-project.org")'

## Download & Install R-package 'sybilSBML'
wget https://cran.r-project.org/src/contrib/Archive/sybilSBML/sybilSBML_3.1.2.tar.gz
R CMD INSTALL --configure-args=" \
--with-sbml-include=$CONDA_PREFIX/include \
--with-sbml-lib=$CONDA_PREFIX/lib" sybilSBML_3.1.2.tar.gz
rm sybilSBML_3.1.2.tar.gz

## Download reference sequence data
bash ./src/update_sequences.sh

##### Note: updated development version is installed in a new environment called gapes-dev2

# Suggested Workflow:

## Reaction & Pathway prediction
/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq find -b 200 -p all -t Bacteria -m Bacteria Laura1_bin.full.78.fa

## Transporter prediction
/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq find-transport -b 200 Laura1_bin.full.78.fa

## Draft network reconstruction
/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq draft -r Laura1_bin.full.78-all-Reactions.tbl -t Laura1_bin.full.78-Transporter.tbl -b Bacteria -c Laura1_bin.full.78.fa -p Laura1_bin.full.78-all-Pathways.tbl -u 200 -l 100

## gapfill/growth medium prediction
##### this step generates a medium file, to model growth on a sole carbon source, remove alternate carbon sources from the automatically generated medium file

/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq medium -m Laura1_bin.full.78-draft.RDS -p Laura1_bin.full.78-all-Pathways.tbl -c "cpd03778:10"

## Gap-filling
/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq fill -m Laura1_bin.full.78-draft.RDS -n Laura1_bin.full.78-medium.csv -c Laura1_bin.full.78-rxnWeights.RDS -g Laura1_bin.full.78-rxnXgenes.RDS -b 100

## Adapting for growth on TPA
/home/lgschaer/old/PacBio/New_Gapseq/gapseq/./gapseq adapt -m Laura1_bin.full.78.RDS -w cpd03778:TRUE -c Laura1_bin.full.78-rxnWeights.RDS -g Laura1_bin.full.78-rxnXgenes.RDS -b Laura1_bin.full.78-all-Reactions.tbl
