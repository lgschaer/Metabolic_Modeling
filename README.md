# Metabolic Modeling: Gapseq Workflow
##### Updated 12/12/2022 by Laura Schaerer

# Installation: 
##### https://gapseq.readthedocs.io/en/latest/install.html

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
/path/to/gapseq/directory/./gapseq find -b 200 -p all -t Bacteria -m Bacteria <Bin_Name>.fa

## Transporter prediction
/path/to/gapseq/directory/./gapseq find-transport -b 200 <Bin_Name>.fa

## Draft network reconstruction
/path/to/gapseq/directory/./gapseq draft -r <Bin_Name>-all-Reactions.tbl -t <Bin_Name>-Transporter.tbl -b Bacteria -c <Bin_Name>.fa -p <Bin_Name>.tbl -u 200 -l 100

## gapfill/growth medium prediction
##### this step generates a medium file containing terephthalate (cpd03778), max flux = 10
##### to model growth on a sole carbon source, remove alternate carbon sources from the automatically generated medium file
##### to model growth on another compound, change cpd03778 to cpdXXXXX (compound IDs can be found on metacyc or in gapseq documentation)
/path/to/gapseq/directory/./gapseq medium -m <Bin_Name>-draft.RDS -p <Bin_Name>-all-Pathways.tbl -c "cpd03778:10"

## Gap-filling
/path/to/gapseq/directory/./gapseq fill -m <Bin_Name>-draft.RDS -n <Bin_Name>-medium.csv -c <Bin_Name>-rxnWeights.RDS -g <Bin_Name>-rxnXgenes.RDS -b 100

## Adapting for growth on TPA
/path/to/gapseq/directory/./gapseq adapt -m <Bin_Name>.RDS -w cpd03778:TRUE -c <Bin_Name>-rxnWeights.RDS -g <Bin_Name>-rxnXgenes.RDS -b <Bin_Name>-all-Reactions.tbl
