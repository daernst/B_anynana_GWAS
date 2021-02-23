## Call SNPs and indels with HaplotypeCaller

### Submit parallel jobs to an HPC cluster with the following code:

```
#!/bin/bash

### Ensure scaffold "INTERVALS" and "SCAFFOLD" range overlap ###
for SAMPLE in 003 004 005 006 012 013 026 029 030 031 032 033 038 039 041 042 048 071 072 073 074 075 084 085 086 087 088 089 092 093 094 095 098 099 100A 100B 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 157 158 159

do

    qsub -v SAMPLE=$SAMPLE,INTERVALS='301to400' haplotypecaller_script.pbs

done
```

### Run HaplotypeCaller (this is the haplotypecaller_script.pbs script):

```
#!/bin/bash

### ADD SCHEDULER HEADER HERE ###


### Change directories to scratch space ###
cd /scratch/${PBS_JOBID}


### Load required module ###
module load java


### Run HaplotypeCaller ###
/share/apps/bioinformatics/GATK/gatk-4.0.0.0/gatk \
HaplotypeCaller \
    -R /home/daernst/GWAS_Dave/Reference_genome/Bicyclus_anynana_v1.2_-_scaffolds.fa \
    -I /home/daernst/GWAS_Dave/Processed_BAM_files/BAN${SAMPLE}_sorted_filtered_fixMate_SH_RG_soortCoord_markDups.bam \
    -O BAN${SAMPLE}_${INTERVALS}.g.vcf \
    --min-pruning 5 \
    --verbosity ERROR \
    -ERC GVCF \
    --heterozygosity 0.005 \
    -L /home/daernst/GWAS_Dave/interval_files/${INTERVALS}.intervals


### Move output files back to appropriate directory ###
mv *.g.vcf /home/daernst/GWAS_Dave/GVCF_files
```
