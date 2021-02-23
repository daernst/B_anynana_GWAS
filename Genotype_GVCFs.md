## Perform joint genotyping with GenotypeGVCFs

### Submit parallel jobs to an HPC cluster with the following code:

```
#!/bin/bash

### Ensure scaffold "INTERVALS" and "SCAFFOLD" range overlap ###
for SCAFFOLD in {00201..00300}

do

    qsub -v SCAFFOLD=$SCAFFOLD,INTERVALS='201to300' genotype_GVCFs.pbs

done
```

### Genotype GVCF files (this is the genotype_GVCFs.pbs script):

```
#!/bin/bash

### ADD SCHEDULER HEADER HERE ###


### Create variable for Bicyclus scaffold files directory ###
DIRB=/home/daernst/GWAS_Dave/Reference_genome


### Create variable for Bicyclus GVCF files directory ###
DIRV=/home/daernst/GWAS_Dave/GVCF_files


### Change directories to scratch space ###
cd /scratch/${PBS_JOBID}


### Make temporary directories ###
mkdir tmp_DB
mkdir tmp_GenotypeGVCFs


### Load required module ###
module load java


### Consolidate GVCF files for single scaffold ###
/share/apps/bioinformatics/GATK/gatk-4.0.0.0/gatk \
GenomicsDBImport \
    -V ${DIRV}/BAN003_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN004_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN005_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN006_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN012_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN013_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN026_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN029_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN030_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN031_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN032_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN033_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN038_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN039_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN041_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN042_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN048_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN071_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN072_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN073_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN074_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN075_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN084_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN085_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN086_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN087_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN088_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN089_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN092_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN093_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN094_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN095_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN098_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN099_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN100A_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN101_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN102_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN103_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN104_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN105_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN106_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN107_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN108_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN109_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN110_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN111_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN112_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN113_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN114_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN115_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN116_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN117_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN118_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN119_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN120_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN121_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN122_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN123_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN124_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN125_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN126_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN127_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN128_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN129_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN130_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN131_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN132_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN133_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN134_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN135_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN136_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN137_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN138_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN139_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN140_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN141_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN142_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN143_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN144_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN145_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN157_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN158_${INTERVALS}.g.vcf \
    -V ${DIRV}/BAN159_${INTERVALS}.g.vcf \
    --genomicsdb-workspace-path /scratch/${PBS_JOBID}/BANY${SCAFFOLD}_100A_database \
    --intervals BANY${SCAFFOLD} \
    --reader-threads 10 \
    --TMP_DIR /scratch/${PBS_JOBID}/tmp_DB


### Genotype consolidated GVCF files for single scaffold ###
/share/apps/bioinformatics/GATK/gatk-4.0.0.0/gatk --java-options "-Xmx10g" \
GenotypeGVCFs \
    -R ${DIRB}/Bicyclus_anynana_v1.2_-_scaffolds.fa \
    -V gendb://BANY${SCAFFOLD}_100A_database \
    -O BANY${SCAFFOLD}_100A.vcf.gz \
    --heterozygosity 0.005 \
    --TMP_DIR /scratch/${PBS_JOBID}/tmp_GenotypeGVCFs \
    -new-qual true


### Filter VCF file ###
/share/apps/bioinformatics/GATK/gatk-4.0.0.0/gatk \
FilterVcf \
    -I BANY${SCAFFOLD}_100A.vcf.gz \
    -O BANY${SCAFFOLD}_100A_filtered.vcf.gz


### Move output files to appropriate directory ###
mv BANY${SCAFFOLD}_100A* /home/daernst/GWAS_Dave/new-qual
```
