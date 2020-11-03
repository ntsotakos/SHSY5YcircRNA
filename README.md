# SHSY5YcircRNA
#!/bin/bash

# Make a directory and stage our data

mkdir -p /scratch/project-script/data
DATADIRECTORY=/scratch/project-script/data

# Need to import sra files and convert to fastq files
cd $DATADIRECTORY
cut -f1 Pezzini-SraRunTable >> AccessionList.txt

while read line;
do prefetch $line < AccessionList.txt;
done

mkdir $DATADIRECTORY/sra
cp /????$/*.sra $DATADIRECTORY/sra

mkdir $DATADIRECTORY/fastq
cd $DATADIRECTORY/fastq
for file in $DATADIRECTORY/sra/*sra
do parallel-fastq-dump --gzip --threads 8 --outdir . -s $file
done

# Make a directory for our analysis
mkdir -p /scratch/project-script/analyses
ANALYSISDIR=/scratch/project-script/analyses

# Use Docker container to do fastqc
docker run -v $DATADIRECTORY:/work quay.io/biocontainers/fastqc:0.11.7--4 fastqc /work/????$

# Move results to analyses directory
mkdir -p $ANALYSISDIR/fastqc
mv $DATADIRECTORY/*fastqc* $ANALYSISDIR/fastqc

# Use Docker container to run Trimmomatic
FORWARD=$1
REVERSE=$2

docker run -v $DATADIRECTORY:/work quay.io/biocontainers/trimmomatic:0.39--1 trimmomatic PE -threads 72 $FORWARD $REVERSE\
    paired_forward.fastq.gz unpaired_forward.fastq.gz\
    paired_reverse.fastq.gz unpaired_reverse.fastq.gz\
    ILLUMINACLIP:/usr/share/trimmomatic/TruSeq2-PE.fa:2:30:10\
    SLIDINGWINDOW:4:20\
    HEADCROP:10

# Move results to analyses directory
mkdir -p $ANALYSISDIR/trimmomatic
mv /????$/*fastq.gz $ANALYSISDIR/trimmomatic

# make directory to store HISAT2 results
mkdir -p $ANALYSISDIR/HISAT2
cd $ANALYSISDIR/HISAT2

# First we need to make index files and we can use docker to do that
# reference.in= A comma-separated list of FASTA files containing the reference sequences to be aligned to. Here I am assuming reference file/files in DATADIRECTORY

docker run -v $DATADIRECTORY:/work quay.io/biocontainers/hisat2:2.2.0--py36hf0b53f7_4 hisat2-build $DATADIRECTORY/reference.in ht2_indx

# Use Docker container to do HISAT2
docker run -v $DATADIRECTORY:/work quay.io/biocontainers/hisat2:2.2.0--py36hf0b53f7_4 hisat2 -x ht2-idx -1 $ANALYSISDIR/trimmomatic/paired_forward.fastq.gz -2 $ANALYSISDIR/trimmomatic/paired_reverse.fastq.gz -U $ANALYSISDIR/trimmomatic/unpaired_forward.fastq.gz,$ANALYSISDIR/trimmomatic/unpaired_reverse.fastq.gz -S result.sam



