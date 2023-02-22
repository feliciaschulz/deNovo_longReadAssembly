# De Novo Long Read Assembly

## Git Repository
### Initialising a Git repository and linking it to GitHub

```bash
git init
git remote add origin # origin address
git fetch
git pull origin main
```

## Downloading the data
### Unzipping the data
```bash
gunzip
```

### Adding the data to gitignore
```bash
echo ".fastq" >> .gitignore
```

### Investigating the data
```bash
less SRR13577846.fastq
cat SRR13577846.fastq | grep -c + #117525
cat SRR13577846.fastq | wc -l #470100
```

## FastQC
### Preparation
```bash
mkdir FastQC
cd ..
conda activate fastqc
fastqc -v # Check version
```
The FastQC version that was used is v0.11.9.

### Fastqc
```bash
fastqc -o FastQC Data/SRR13577846.fastq
```
The quality of the data seems good overall. In the per base sequence content graph, at the very end, it seemed like the sequence only consisted of Gs. Otherwise, the graphs seemed good.

## Genome Assembly
For the genome assembly, the long-read assembler Raven was chosen. In various different sources, it is stated that this assembler has good accuracy (though some weaknesses) and is very fast. Therefore, for the sake of finding whether this fast assembler can still compete with others such as Hifi, this one was chosen.

Sources: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6966772/
https://www.frontiersin.org/articles/10.3389/fmicb.2022.796465/full
https://www.biorxiv.org/content/10.1101/2020.08.07.242461v2.full


### Installation with conda
```bash
conda create -n raven
conda activate raven
conda install -c bioconda raven-assembler
raven --version
```
Raven was used with version 1.8.1

### Running raven
```bash
raven ../Data/SRR13577846.fastq
```
This took 2919.665494 seconds (49 minutes) when using only one thread.

### Saving the output
```bash
raven ../Data/SRR13577846.fastq --resume > SRRoutput.fa
```

## QUAST
### QUAST version
```bash
conda activate quast
quast -v
```
QUAST was run using version 5.2.0

### Running QUAST with conda
```bash
quast -o Quast Assembly/SRRoutput.fasta
```

