# SHSY5YcircRNA
#!/bin/bash

# Make a directory and stage our data

mkdir -p /scratch/project-script/data
DATADIRECTORY=/scratch/project-script/data

#Need to import fastq file

#Make a directory for our analysis
mkdir -p /scratch/project-script/analyses
ANALYSISDIR=/scratch/project-script/analyses

#Use Docker container to do fastqc
docker run -v $DATADIRECTORY:/work quay.io/biocontainers/fastqc:0.11.7--4 fastqc /work/????$

#move results to analyses directory
mkdir -p $ANALYSISDIR/fastqc
mv $DATADIRECTORY/*fastqc* $ANALYSISDIR/fastqc


