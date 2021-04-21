# pixy_flow

## 1. input files processing
### the $inputvcf are from raw GATK output (chromosome-specific)

1. Filter

```
vcftools --gzvcf $inputvcf \
--max-maf 0 --minQ 30 --remove-indels --max-missing 0.5 --min-meanDP 10  \
--recode --recode-INFO-all --stdout | bgzip -c > snp_invariants_${chrm}.vcf.gz

tabix snp_invariants_${chrm}.vcf.gz

vcftools --gzvcf $inputvcf \
--mac 1 --min-alleles 2 --max-alleles 2 --max-missing 0.5 --min-meanDP 10 \
--recode --recode-INFO-all --remove-indels --stdout | bgzip -c > snp_variants_${chrm}.vcf.gz

tabix snp_variants_${chrm}.vcf.gz
```

2. Combine invariants and variants
```
bcftools concat --allow-overlaps \
snp_invariants_${chrm}.vcf.gz snp_variants_${chrm}.vcf.gz \
-O z -o allsites_filtered_${chrm}.vcf.gz

tabix allsites_filtered_${chrm}.vcf.gz
```

3. Run pixy 
```
#1
pixy --stats pi \
--vcf allsites_filtered_${chrm}.vcf.gz \
--chromosomes ${chrm} \
--window_size 20000 \
--populations pop.pixy.txt \
--output_prefix ${chrm}_out_pi

#2
pixy --stats pi \
--vcf allsites_filtered_${chrm}.vcf.gz \
--chromosomes ${chrm} \
--interval_start 60000001 \
--interval_end 80000000 \
--window_size 20000 \
--populations pop.pixy.txt \
--output_prefix ${chrm}_out_pi

#3
pixy --stats fst \
--vcf allsites_filtered_${chrm}.vcf.gz \
--chromosomes ${chrm} \
--window_size 20000 \
--populations pop.pixy.txt \
--output_prefix ${chrm}_out_fst

```
