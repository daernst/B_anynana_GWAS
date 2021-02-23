## Align reads to *B. anynana* reference genome and prepare BAM for GATK

### Submit parallel jobs to an HPC cluster with the following code:

```
#!/bin/bash

for SAMPLE in 003 004 005 006 012 013 026 029 030 031 032 033 038 039 041 042 048 071 072 073 074 075 084 085 086 087 088 089 092 093 094 095 098 099 100A 100B 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 157 158 159

do

    qsub -v SAMPLE=$SAMPLE create_and_prep_BAM_for_gatk.pbs

done
```

### Align reads and prepare BAM (this is the create_and_prep_BAM_for_gatk.pbs script):

```
#!/bin/bash

### ADD SCHEDULER HEADER HERE ###


### Load required modules ###
module load java


### Change directories to scratch space ###
cd /scratch/${PBS_JOBID}


### Align reads to Bicyclus reference genome ###
echo Aligning BAN${SAMPLE} reads to reference genome...

/home/daernst/GWAS_Dave/Bioinformatics_programs/bowtie2-2.1.0-linux-x86_64/bowtie2-2.1.0/bowtie2 \
    --very-sensitive-local \
    -p 24 \
    -x /home/daernst/GWAS_Dave/Reference_genome/Bicyclus_anynana_v1.2_-_scaffolds \
    -1 /home/daernst/GWAS_Dave/Trimmed_concatenated_runs/BAN${SAMPLE}_R1_trimmed_paired.fq.gz \
    -2 /home/daernst/GWAS_Dave/Trimmed_concatenated_runs/BAN${SAMPLE}_R2_trimmed_paired.fq.gz \
    -S /home/daernst/GWAS_Dave/Bowtie2_alignment_SAM_files/BAN${SAMPLE}_alignment.sam

echo BAN${SAMPLE} alignment complete.


### Sort the SAM file by queryname and create BAM file ###
echo Sorting BAN${SAMPLE} SAM by queryname and creating BAM...

mkdir tmp

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
SortSam \
    I=/home/daernst/GWAS_Dave/Bowtie2_alignment_SAM_files/BAN${SAMPLE}_alignment.sam \
    O=BAN${SAMPLE}_sorted.bam \
    SORT_ORDER=queryname \
    TMP_DIR=`pwd`/tmp

echo SAM sorting and BAM creation for BAN${SAMPLE} complete.


### Filter out unmapped reads from BAM file ###
echo Filtering out unmapped reads from BAN${SAMPLE} BAM file...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
FilterSamReads \
    I=BAN${SAMPLE}_sorted.bam \
    O=BAN${SAMPLE}_sorted_filtered.bam \
    FILTER=includeAligned

echo BAN${SAMPLE} BAM filtering complete.


### Fix the mate-pair information in the BAM file (this ensures that all mate-pair information is in sync between each read and its mate pair) ###
echo Fixing mate-pair information in BAN${SAMPLE} BAM file...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
FixMateInformation \
    I=BAN${SAMPLE}_sorted_filtered.bam \
    O=BAN${SAMPLE}_sorted_filtered_fixMate.bam

echo BAN${SAMPLE} mate-pair fixing complete.


### Replace the SAM header of the BAM file ###
echo Replacing SAM header for BAN${SAMPLE}...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
ReplaceSamHeader \
    I=BAN${SAMPLE}_sorted_filtered_fixMate.bam \
    HEADER=/home/daernst/GWAS_Dave/Reference_genome/Bicyclus_anynana_v1.2_-_scaffolds.dict \
    O=BAN${SAMPLE}_sorted_filtered_fixMate_SH.bam

echo SAM header replacement for BAN${SAMPLE} complete.


### Add or replace read groups in the BAM header (this assigns all the reads in a file to a single new read-group, as required by GATK) ###
echo Adding or replacing read groups in BAN${SAMPLE} BAM header...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
AddOrReplaceReadGroups \
    I=BAN${SAMPLE}_sorted_filtered_fixMate_SH.bam \
    O=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG.bam \
    RGID=1 \
    RGLB=lib1 \
    RGPL=illumina \
    RGPU=unit1 \
    RGSM=BAN${SAMPLE}

echo Add/replace BAN${SAMPLE} read groups complete.


### Sort the BAM file by coordinate instead of queryname ###
echo Sorting BAN${SAMPLE} BAM by coordinate...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
SortSam \
    I=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG.bam \
    O=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_sortCoord.bam \
    SORT_ORDER=coordinate

echo Coordinate sorting for BAN${SAMPLE} complete.


### Mark and remove duplicate reads in the BAM file ###
echo Marking and removing duplicate reads in BAN${SAMPLE} BAM file...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
MarkDuplicates \
    I=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_sortCoord.bam \
    O=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_soortCoord_markDups.bam \
    M=BAN${SAMPLE}_dup_metrics.txt \
    REMOVE_DUPLICATES=true

echo Duplicate read marking and removal for BAN${SAMPLE} complete.


### Build an index for the BAM file ###
echo Building index for BAN${SAMPLE} BAM file...

java -jar /home/daernst/GWAS_Dave/Bioinformatics_programs/picard.jar \
BuildBamIndex \
    I=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_soortCoord_markDups.bam \
    O=BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_soortCoord_markDups.bam.bai

echo BAN${SAMPLE} BAM index building complete.


### Move processed bam files to home directory ###
echo Moving processed BAN${SAMPLE} BAM files to home directory...

mv BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_soortCoord_markDups.bam* /home/daernst/GWAS_Dave/Processed_BAM_file

echo File moving for BAN${SAMPLE} complete.
```
