# PCFinder: A computational pipeline to identify polyploid cancer cells in single cell RNA-seq data

Citation: TBD

All analysis scripts can be found in src folder. A WDL pipeline with individual scripts is also provided under src/pipeline, along with example JSON, Dockerfiles, and Config files as well as submission scripts to be run on HPC (e.g Compute1 at WashU). These scripts can also be run individually in the following order: 1. ) pcfinder_cnv_estimation.R 2.) pcfinder_run_models.py. This pipeline involes fast CNV-inference of scRNA-seq count data using established methods (CopyKat), followed by running 5 ML/DL classifiers on various features associated with polyploidization to output the predicted probability of PC for each cell type. Example RDS files containing Seurat objects for each dataset used in the publication can be found under the data folder for replication.  

Usage:
  Rscript pcfinder_cnv_estimation.R \
    --seurat      /path/to/object.rds \
    --epi_count   4000 \
    --tme_count   4000 \
    --seed        42 \
    --celltype_col coarse_ano \
    --epi_label   "Epi_Neuroendo" \
    --out_dir     /path/to/output/ \
    [--spikein    /path/to/spikein.rds] \
    [--spikein_malignancy_col  malignancy] \
    [--spikein_malignancy_val  "Non-Malignant"] \
    [--spikein_id_col  ID] \
    [--spikein_id_val  LE] \
    [--mt_col     percent.mt] \
    [--ncores     8]
Assume a barcode column in metadata labeled "cell" to uniquely identify cells

Required arguments:
  --seurat          Path to input Seurat RDS file
  --epi_count       Number of epithelial cells to downsample
  --tme_count       Number of TME / reference cells to downsample
  --seed            Random seed for reproducibility
  --celltype_col    Metadata column used to identify cell types
  --epi_label       Value in --celltype_col that marks epithelial/tumor cells
  --out_dir         Directory for all output files

Optional arguments:
  --spikein                   Path to an external spike-in Seurat RDS used as
                              the diploid reference. If omitted, non-epithelial
                              cells from the main object are used instead.
  --spikein_malignancy_col    Metadata column in spike-in to filter on
                              (default: malignancy)
  --spikein_malignancy_val    Value to keep in that column
                            (default: Non-Malignant)
  --mt_col                    Metadata column for mitochondrial percent
                              (default: percent.mt)
  --ncores                    Cores passed to copykat (default: 8)
  
  

Usage: 
  pcfinder_run_models.py [-h] [--out OUT] [--model_dir MODEL_DIR] [--features FEATURE [FEATURE ...]] [--drop_na] [--cell_id_col COL] input_csv

Run trained PACCs classifiers on new data

positional arguments:
  input_csv             Input CSV file with features (output of pcfinder_cnv_estimation.R)

options:
  -h, --help            show this help message and exit
  --out OUT             Output CSV file for predictions (default: paccs_predictions.csv)
  --model_dir MODEL_DIR
                        Directory containing saved .pkl models and scaler.pkl (default: models/)
  --features FEATURE [FEATURE ...]
                        Feature column names to use. Trailing digits from Seurat's
                        Default: ['Percent.MT', 'S.Score', 'cnv_total', 'nCount_RNA', 'nFeature_RNA', 'G2M.Score']
  --drop_na             Drop rows with any missing values before inference (default: True)
  --cell_id_col COL     Column to use as a cell identifier in the output (optional)
  

WDL pipeline from Command Line:
java -Xms4g -Xmx12g \
       -Dconfig.file=cromwell_compute1.conf \
       -jar cromwell-86.jar \
       run pcfinder.wdl \
       --inputs pcfinder.json

WDL pipeline from HPC (Compute1 at WashU, see subtmit_wdl.sh script)
bsub \
  -J pcfinder \
  -G compute-christophermaher \
  -g /saha.d/max100 \
  -q general \
  -n 4 \
  -R 'select[mem>16000] rusage[mem=16000]' \
  -M 16000000 \
  -oo Logs/pcfinder.out \
  -eo Logs/pcfinder.err \
  -a 'docker(openjdk:11.0.11-jdk-slim)' /usr/local/openjdk-11/bin/java -Xms4g -Xmx12g \
       -Dconfig.file=cromwell_compute1.conf \
       -jar cromwell-86.jar \
       run "${PROJECT_DIR}"/R/crpc_analysis/pcfinder.wdl \
       --inputs pcfinder.json
  
