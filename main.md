Following tutorial at https://www.genomatics.net/guides/gatk-best-practices-installation-guide

I first copied my refs that i used last time at grch38 and with all indexing to folder ref inside sftp://dell_815@10.1.131.145/media/ds1821p_III/aserikzhan/variant_calling/
(turns out its not actually necessary but its fine)

I will do it isolatedin docker container:
```bash
docker pull broadinstitute/gatk
```

Running:
```bash
docker run -it -v /media/ds1821p_III/aserikzhan/variant_calling/:/home broadinstitute/gatk bash
cd ../home/
```

Installations (failed without docker)
```bash
# Create isolated environment
conda create -n gatk-env -c bioconda -c conda-forge \
  gatk4=4.5.0.0 \
  bwa=0.7.17 \
  samtools=1.17 \
  picard=3.1.0 \
  python=3.10

# Activate environment
conda activate gatk-env

# Verify installation
gatk --version
# Expected output: The Genome Analysis Toolkit (GATK) v4.5.0.0

# Create reference directory
mkdir -p ~/genomics/references
cd ~/genomics/references

# Download human reference genome (GRCh38/hg38)
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.fasta
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.fasta.fai
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.dict

# Download known sites (for BQSR)
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.dbsnp138.vcf
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.known_indels.vcf.gz
wget https://storage.googleapis.com/genomics-public-data/resources/broad/hg38/v0/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz

# Index VCF files
gatk IndexFeatureFile -I Homo_sapiens_assembly38.dbsnp138.vcf
gatk IndexFeatureFile -I Homo_sapiens_assembly38.known_indels.vcf.gz
gatk IndexFeatureFile -I Mills_and_1000G_gold_standard.indels.hg38.vcf.gz

# Create BWA index
bwa index Homo_sapiens_assembly38.fasta
