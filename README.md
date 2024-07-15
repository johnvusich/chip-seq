# chip-seq
Pipeline to analyze ChIP-seq data on the HPCC using SLURM.

## Steps
1. Create a directory for analysis.
2. Prepare a samplesheet table in csv format. (ex: samplesheet.csv)
3. Make a Nextflow config file.
4. Download the reference genome and file in your directory.
5. Write a bash script to run the pipeline using SLURM.
6. Run the pipeline from your ChIP-seq directory.

## Create a directory
Login to your HPCC account using OnDemand. Navigate to your home directory by clicking 'Files' in the navigation bar. Select 'Home Directory'.

Create a directory for your analysis by clicking 'New Directory'. Name your directory (ex: chipseq). Navigate to the newly created ChIP-seq directory.

Make sure to upload your data (FASTQ files) into this directory.

## Make a samplesheet table
In your ChIP-seq directory, click 'New File'. Name the file 'samplesheet.csv'. Click the `⋮` symbol and select edit. Create the samplesheet table FOR YOUR DATA. Below is a template:
```
sample,fastq_1,fastq_2,antibody,control
WT_BCATENIN_IP_REP1,BLA203A1_S27_L006_R1_001.fastq.gz,,BCATENIN,WT_INPUT
WT_BCATENIN_IP_REP2,BLA203A25_S16_L001_R1_001.fastq.gz,,BCATENIN,WT_INPUT
WT_INPUT_REP1,BLA203A6_S32_L006_R1_001.fastq.gz,,,
WT_INPUT_REP2,BLA203A30_S21_L001_R1_001.fastq.gz,,,
```
Note: the samplesheet above is an example to show the required format. You will still need to make a samplesheet FOR YOUR DATA.

Save the samplesheet.csv file and return to your ChIP-seq directory.

## Make a Nextflow configuration file to use SLURM as the process executor
In your ChIP-seq directory, click `New File`. Name the file 'nextflow.config'. Click the `⋮` symbol and select edit. Create the Nextflow config file:
```
process {
    executor = 'slurm'
}
```
Save the nextflow.config file and return to your ChIP-seq directory.

## Find your reference genome in ICER common-data OR download the reference genome and gtf files in your directory
The following organisms have updated reference genomes and gtf/gff3 files in common-data: human, mouse, rat, zebrafish, and Arabidopsis. The paths to those files can be found [here](https://github.com/johnvusich/reference-genomes).

In your directory, click `Open in Terminal` to enter a development node. Download the most recent genome primary assembly and gtf for the organism of interest from [Ensembl](https://ensembl.org/). Follow the instructions [here](https://github.com/johnvusich/reference-genomes) to download the reference genome for your organism of interest. This may take more than a few minutes. For example:
```
wget https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz

wget https://ftp.ensembl.org/pub/release-108/gtf/homo_sapiens/Homo_sapiens.GRCh38.108.gtf.gz
```
Note: If you do not download the genome and gtf for the organism that the RNA-seq data is derived from, this pipeline will not work correctly.

## Write a bash script to run the pipeline using SLURM
In your ChIP-seq directory, click `New File`. Name the file 'run_chipseq.sb;. Write the bash script, using #SBATCH directives to set resources, for example:
```
#!/bin/bash

#SBATCH --job-name=$jobname_chipseq
#SBATCH --time=24:00:00
#SBATCH --mem=24GB
#SBATCH --cpus-per-task=8

cd $HOME/chipseq
module load Nextflow/23.10.0

nextflow pull nf-core/chipseq
nextflow run nf-core/chipseq -r 2.0.0 --input ./samplesheet.csv  -profile singularity --outdir ./chipseq_results --fasta ./Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz --gtf ./Homo_sapiens.GRCh38.108.gtf.gz -work-dir $SCRATCH/chipseq_work -c ./nextflow.config
```
Save the run_chipseq.sb file and return to your ChIP-seq directory.

## Run the pipeline
In your ChIP-seq directory, click `Open in Terminal` to enter a development node. Run jobs on the SLURM cluster:
```
sbatch run_chipseq.sb
```
Check the status of your pipeline job:
```
squeue -u $username
```
Note: replace $username with your username
