# De Novo Long Read Assembly
In this assembly we will first investigate the data, then assemble the reads and then run more analyses on the assembled genome. The structure of this README file is the following:  
1. Git Repository  
2. Downloading the data  
3. FastQC  
4. Genome Assembly
5. QUAST  
6. BUSCO

The data used here is a fastq file consisting of 117,525 reads. The sequenced reads are from the organism Saccharomyces cerevisiae. The data can be found and downloaded from here: https://trace.ncbi.nlm.nih.gov/Traces/?view=run_browser&acc=SRR13577846&display=metadata



## Git Repository
### Initialising a Git repository and linking it to GitHub

First, a Git repository must be created and linked to the private remote repository https://github.com/feliciaschulz/deNovo_longReadAssembly.git.  
By doing this, a reproducible workflow can be ensured.

```bash
git init
git remote add origin # <origin address>
git fetch
git pull origin main
```

## Downloading the data
As stated above, the data was downloaded from https://trace.ncbi.nlm.nih.gov/Traces/?view=run_browser&acc=SRR13577846&display=metadata.
### Unzipping the data
```bash
gunzip
```

### Adding the data to gitignore
This fastq file is very large and it will not be possible to commit it to the remote repository as GitHub has a file size limit. On top of that, it is not considered good practice to have all datasets on a Git repository, only small sample sets.
```bash
echo ".fastq" >> .gitignore
```

### Investigating the data
```bash
less SRR13577846.fastq
cat SRR13577846.fastq | grep -c + #117525
cat SRR13577846.fastq | wc -l #470100
```
The data has 117252 reads.

## FastQC
FastQC is a bioinformatics tool which provides quality control for raw sequence data.
### Preparation
```bash
mkdir FastQC
cd ..
conda activate fastqc
fastqc -v # Check version
```
The FastQC version that was used is v0.11.9.

### FastQC analysis
```bash
fastqc -o FastQC Data/SRR13577846.fastq
```
The quality of the data seems good overall. In the per base sequence content graph, at the very end, it seemed like the sequence only consisted of Gs. When investigating the data further, it becomes clear that this is the case because the longest read has a G at the end, and it is the only one which falls into the uppermost length category on this FastQC graph. This is why suddenly it looks like at that base pair index, there are 100% Gs. Hence, this is not a problem whatsoever. The other graphs all look very good.

## Genome Assembly
For the genome assembly, the long-read assembler Raven was chosen. In various different sources, it is stated that this assembler has good accuracy (though some weaknesses) and is very fast. Therefore, for the sake of finding whether this fast assembler can still compete with  others such as Hifi, this one was chosen.

Sources:  
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6966772/  
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
With raven, the output is not saved to an output file immediately. Therefore, you can either save it immediately when initially running, or save it with an additional command afterwards, just like here. 
```bash
raven ../Data/SRR13577846.fastq --resume > SRRoutput.fasta
```

## QUAST
QUAST is a bioinformatics tool which evaluates the quality of genome assemblies.
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
The quast summarises the assembly in the following way: There are 22 contigs, the largest being 1495363 base pairs long. The total length of the contigs is 12051270bp. The N50 is 802231, which means that the smallest contig which makes up for 50% of the genome is 802231 base pairs long. The GC content is 38.32%.


### QUAST with reference genome
The reference genome was obtained from https://www.ncbi.nlm.nih.gov/data-hub/taxonomy/4932/ .
The results for this can be found in the folder reference_Quast.

```bash
quast -r GCF_000146045.2_R64_genomic.fna -t 1 ../../Assembly/SRRoutput.fasta
```
The specification -t 1 had to be used because of an error, but did not slow down the process.

The QUAST analysis with the reference genome showed that there were 211.81 mismatches per 100 kbp. The length of the misassembled contigs is 11372283bp. The total aligned length is 11956409. There are 21.6 indels per 100 kbp. The NGA50 is 267325bp. NGA50 is similar to N50, just that it counts the length of the longest aligned sequences rather than the length of the contigs themselves.


### Adding the reference to gitignore
The reference genome is likely too large to push to the remote repository and is therefore added to .gitignore
```bash
echo "GCF*" >> .gitignore
```

## BUSCO
### BUSCO versions
```bash
busco -v
```
The version of BUSCO that was used was 5.4.4. BUSCO itself uses the following dependencies and versions:

hmmsearch 3.1  
bbtools 39.01  
prodigal 2.6.3  
metaeuk 6.a5d39d9

This information can be found in the busco output files such as short_summary.genetic.eukaryota_odb10.busco_output.txt.

### Running BUSCO with conda
First, BUSCO is installed using conda and a separate environment is created for it.
```bash
conda install -c conda-forge -c bioconda busco -y
```

Then, BUSCO can be run with the assembly data.
```bash
mkdir busco
cd busco
busco -i ../Assembly/SRRoutput.fasta -o busco_output -m genome
```
The BUSCO analysis out put shows 2137 total BUSCO groups searched. Among this are 2120 complete BUSCOS (99.6%), 2 fragmented BUSCOs (0.1%) and 6 missing BUSCOS (0.3%). A complete busco score above 95% is considered good, therefore these are good results. This suggests that there are in fact highly conserved genes in the assembly.






