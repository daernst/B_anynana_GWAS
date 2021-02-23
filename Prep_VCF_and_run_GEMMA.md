## Prepare VCF files for association analysis and run GEMMA:

```
#!/bin/bash

### ADD SCHEDULER HEADER HERE ###


echo Job start time : `date +"%H:%M:%S"`


### Change directories to scratch space ###
cd /scratch/${SLURM_JOB_ID}


### Load necessary modules ###
echo Loading required modules...
module load vcftools tabix java perl
echo Done.


### Start loop ###
echo Starting pipeline...
echo Time : `date +"%H:%M:%S"`
for SCAFFOLD in {00001..02000};

do

    echo XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX STARTING ANALYSIS FOR SCAFFOLD BANY${SCAFFOLD} XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    echo Time : `date +"%H:%M:%S"`

    echo Copying BANY${SCAFFOLD}_100A_filtered.vcf.gz to scratch space...
    echo Time : `date +"%H:%M:%S"`
    cp /home/daernst/GWAS_Dave/TEST_SPACE/new-qual/BANY${SCAFFOLD}_100A_filtered.vcf.gz .
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Fix ploidy issue (fix by replacing single "." GT with "./.") ###
    echo Fixing ploidy for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/GWAS_Dave/Bioinformatics_programs/bcftools-1.9/bcftools \
        +fixploidy \
        BANY${SCAFFOLD}_100A_filtered.vcf.gz > BANY${SCAFFOLD}_100A_filtered_fixed.vcf.gz
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Get biallelic sites on scaffold of interest ###
    echo Getting biallelic sites for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/GWAS_Dave/Bioinformatics_programs/bcftools-1.9/bcftools view \
        -m 2 \
        -M 2 \
        --output-type z \
        --output-file BANY${SCAFFOLD}_biallelic_1.vcf.gz \
        BANY${SCAFFOLD}_100A_filtered_fixed.vcf.gz
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Check vcf file with PLINK  ###
    echo Checking vcf file missingness with PLINK for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /share/apps/bioinformatics/plink/5.2/plink \
        --vcf BANY${SCAFFOLD}_biallelic_1.vcf.gz \
        --allow-extra-chr \
        --missing \
        --noweb \
        --allow-no-sex \
        --out BANY${SCAFFOLD}_biallelic_1
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Make list of fully missing (100%) individuals ###
    awk '$6 == "1" {print $1}' BANY${SCAFFOLD}_biallelic_1.imiss > BANY${SCAFFOLD}_miss_indiv.txt


    ### Filter out variants with missingness >20% ###
    echo Filtering out fully missing individuals and variants with missingness greater than 20 percent for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    vcftools \
        --gzvcf BANY${SCAFFOLD}_biallelic_1.vcf.gz \
        --remove BANY${SCAFFOLD}_miss_indiv.txt \
        --max-missing 0.8 \
        --recode \
        --out BANY${SCAFFOLD}_biallelic
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Create index for biallelic vcf ###
    echo Creating index for biallelic vcf for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/GWAS_Dave/Bioinformatics_programs/bcftools-1.9/bcftools view \
        --output-type z \
        --output-file BANY${SCAFFOLD}_biallelic.vcf.gz \
        BANY${SCAFFOLD}_biallelic.recode.vcf
    tabix BANY${SCAFFOLD}_biallelic.vcf.gz
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Check new biallelic vcf file with PLINK  ###
    echo Checking new biallelic vcf file missingness with PLINK for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /share/apps/bioinformatics/plink/5.2/plink \
        --vcf BANY${SCAFFOLD}_biallelic.vcf.gz \
        --allow-extra-chr \
        --missing \
        --noweb \
        --allow-no-sex \
        --out BANY${SCAFFOLD}_biallelic
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ################################
    ### PREP FILES AND RUN GEMMA ###
    echo GEMMA ANALYSIS

    ### Make PLINK from VCF ###
    echo Making PLINK file from vcf for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    vcftools \
        --gzvcf BANY${SCAFFOLD}_biallelic.vcf.gz \
        --plink \
        --out BANY${SCAFFOLD}_biallelic.plink
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Add phenotype data to PLINK file ###
    echo Adding phenotype data to PLINK file for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    awk '{
        if($1=="BAN004" || $1=="BAN005" || $1=="BAN006" || $1=="BAN026" || \
           $1=="BAN048" || $1=="BAN085" || $1=="BAN086" || $1=="BAN094" || \
           $1=="BAN095" || $1=="BAN100A" || $1=="BAN101" || $1=="BAN102" || \
           $1=="BAN103" || $1=="BAN104" || $1=="BAN121" || $1=="BAN123" || \
           $1=="BAN124" || $1=="BAN125" || $1=="BAN126" || $1=="BAN127" || \
           $1=="BAN128" || $1=="BAN129" || $1=="BAN130" || $1=="BAN131" || \
           $1=="BAN132" || $1=="BAN133" || $1=="BAN134" || $1=="BAN012" || \
           $1=="BAN029" || $1=="BAN030" || $1=="BAN032" || $1=="BAN038" || \
           $1=="BAN033" || $1=="BAN039" || $1=="BAN042" || $1=="BAN093" || \
           $1=="BAN003" || $1=="BAN084" || $1=="BAN122" || $1=="BAN087") {
           $6 = 1
           }
        else{
              $6 = 0
             }
        print $0
    }' \
    BANY${SCAFFOLD}_biallelic.plink.ped > \
                            BANY${SCAFFOLD}_biallelic.plink.recode.ped
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Rename map file ###
    echo Renaming map file for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    mv BANY${SCAFFOLD}_biallelic.plink.map \
        BANY${SCAFFOLD}_biallelic.plink.recode.map
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Make PLINK binary files ###
    echo Making PLINK binary files for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /share/apps/bioinformatics/plink/5.2/plink \
        --file BANY${SCAFFOLD}_biallelic.plink.recode \
        --make-bed \
        --noweb \
        --allow-no-sex \
        --out BANY${SCAFFOLD}_biallelic.plink.recode
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Run PCA with PLINK ###
    echo Running PCA with PLINK for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /share/apps/bioinformatics/plink/5.2/plink \
        --bfile BANY${SCAFFOLD}_biallelic.plink.recode \
        --noweb \
        --allow-no-sex \
        --pca 20 \
        --out BANY${SCAFFOLD}_biallelic.plink.recode
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Prepare eigenvector file for GEMMA ###
    echo Time : `date +"%H:%M:%S"`
    echo Preparing eigenvector file for GEMMA for BANY${SCAFFOLD}...
    awk '{$1=1; print $1"\t"$3}' \
        BANY${SCAFFOLD}_biallelic.plink.recode.eigenvec > \
        BANY${SCAFFOLD}_biallelic.plink.recode.eigenvec_1
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Create relatedness matrix ###
    echo Creating relatedness matrix for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/gemma-0.98-linux-static \
        -bfile BANY${SCAFFOLD}_biallelic.plink.recode \
        -gk 1 \
        -miss 0.2 \
        -maf 0.05 \
        -o BANY${SCAFFOLD}_biallelic.plink.recode.relatedness
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Make association file using relatedness matrix ###
    echo Making association file using relatedness matrix for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/gemma-0.98-linux-static \
        -bfile BANY${SCAFFOLD}_biallelic.plink.recode \
        -k ./output/BANY${SCAFFOLD}_biallelic.plink.recode.relatedness.cXX.txt \
        -lmm 4 \
        -miss 0.2 \
        -maf 0.05 \
        -o BANY${SCAFFOLD}_biallelic_miss-10_MAF_0.05
    echo Done.
    echo Time : `date +"%H:%M:%S"`


    ### Make association file using top PC (eigenvector) covariate from PLINK PCA and relatedness matrix ###
    echo Making association file using top principle component and relatedness matrix for BANY${SCAFFOLD}...
    echo Time : `date +"%H:%M:%S"`
    /home/daernst/gemma-0.98-linux-static \
        -bfile BANY${SCAFFOLD}_biallelic.plink.recode \
        -k ./output/BANY${SCAFFOLD}_biallelic.plink.recode.relatedness.cXX.txt \
        -c BANY${SCAFFOLD}_biallelic.plink.recode.eigenvec_1 \
        -lmm 4 \
        -miss 0.2 \
        -maf 0.05 \
        -o BANY${SCAFFOLD}_biallelic_miss-10_MAF_0.05.PCA-top1
    echo Done.
    echo Time : `date +"%H:%M:%S"`

done
```
