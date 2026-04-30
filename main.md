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
conda init
```

In another shell saving new version 
```bash
docker commit 50aa806dd4e0 gatk:v2.0
docker run -it -v /media/ds1821p_III/aserikzhan/variant_calling/:/home gatk:v2.0 bash
cd ../home
```

Continuing:
```bash
conda activate gatk-env

# Verify installation
gatk --version
# Expected output: The Genome Analysis Toolkit (GATK) v4.5.0.0

# Create reference directory
mkdir -p genomics/references
cd /genomics/references

# Download human reference genome (GRCh38/hg38) - the original didn't work so i used ftp
wget ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Homo_sapiens_assembly38.fasta.gz
wget ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Homo_sapiens_assembly38.fasta.fai
wget ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Homo_sapiens_assembly38.dict

# Download known sites (for BQSR)
wget ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/dbsnp_146.hg.vcf.gz
wget ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz

# we'll have dbsnp_146.hg.vcf.gz instead of Homo_sapiens_assembly38.dbsnp138.vcf
# Mills_and_1000G_gold_standard.indels.hg38.vcf.gz = Mills_and_1000G_gold_standard.indels.hg38.vcf.gz stays the same
3 I don't have Homo_sapiens_assembly38.known_indels.vcf.gz so we'll do without it

# stopped here because im waiting for 2 big files to finish downloading; not sure if i need to unzip them first

# Index VCF files
gatk IndexFeatureFile -I dbsnp_146.hg.vcf.gz
gatk IndexFeatureFile -I Mills_and_1000G_gold_standard.indels.hg38.vcf.gz

# Create BWA index
bwa index Homo_sapiens_assembly38.fasta.gz
