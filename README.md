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
```

### Fastqc
```bash
fastqc -o FastQC Data/SRR13577846.fastq
```


