# ROHbyPlink
Visualization and proportion calculation of ROH estimated by Plink.

Here we provide a guide for good practice on data filtering and visualization.

## Data filter
The input data is in **VCF/BCF** format, we suggest three steps of **Sites** filter using **bcftools** and **plink**: 
  - Filter on Genotype Calling
  - Filter within Species/Sub-species
  - Filter for each Individual

## Data and Softwares
  ```
  inputBcf = /path/to/bcf
  bcftools = /path/to/bcftools
  plink = /path/to/plink
  
  ```

### Filter on Genotype Calling
  - Depth minimum 10x
  ```
  depthFilteredBcf = Give_it_a_name_ends_dot_bcf_dot_gz
  $bcftools plugin setGT --threads 10 $inputBcf --include FMT/DP<10 --target-gt q --new-gt . --output-type b --output $depthFilteredBcf
  ```
  - Minimum 3 reads for each allele for heterozygous call
  ```
  hetReadsFilteredBcf = Give_it_a_name_dot_bcf_dot_gz
  $bcftools plugin setGT --threads 10 $depthFilteredBcf --include 'FMT/GT=='het' & (FMT/AD[*:0]<3 | FMT/AD[*:1]<3)' --target-gt q --new-gt . --output-type b --output $hetReadsFilteredBcf
  ```
  - Strand bias: ratio of alt allele+/- & ratio of ref allele+/- less than 10
  ```
  strandBiasFilterdBcf = Give_it_a_name_dot_bcf_dot_gz
  $bcftools filter --exclude 'DP4[2]/DP4[3]>10 | DP4[3]/DP4[2]>10 | DP4[0]/DP4[1]>10 |DP4[1]/DP4[0]>10' --threads 10 $hetReadsFilteredBcf --output-type b --output $strandBiasFilterdBcf
  ```
  - Allelic balance: minimum 25% for each allele
  ```
  allellicBalanceFilterdBcf = Give_it_a_name_dot_bcf_dot_gz
  $bcftools plugin setGT $strandBiasFilterdBcf -t q --include 'FMT/GT=="het" & ((FORMAT/AD[*:0] / (FMT/DP) ) <0.25 | (FMT/AD[*:0] / (FMT/DP)) >0.75)' -n ./. --output-type b --output $allellicBalanceFilterdBcf
  ```

### Filter within Species/Sub-species
  - Minimum maf 5%
  - Minimum alt count 4
  - Maximum 50% missing
  ```
  #put the samples of a species/sub-species into a file: example.txt, each line has one sample ID
  sampleID = example.txt
  speciesFilteredBfile = Give_it_a_name
  $bcftools view $allellicBalanceFilterdBcf -S $sampleID -q 0.05:minor |$bcftools filter --include 'AC>=4' |$bcftools view -O v |$plink --vcf /dev/stdin --allow-extra-chr --chr-set 29 --double-id  --set-missing-var-ids @:# --make-bed --geno 0.5 --out $speciesFilteredBfile
  ```

### Filter for each Individual
  - Remove all missing sites
  ```
  cat $sampleID |while read sample
  do
    grep $sample $speciesFilteredBfile.fam > $sample.keep.fam
    $plink --allow-extra-chr --bfile $speciesFilteredBfile --keep $sample.keep.fam --double-id --geno 0 --make-bed --out $sample --set-missing-var-ids @:#
   done
  ```

## ROH estimation and visualization
  - Estimated ROH by Plink for each individual and visualization of distribution of ROH
  ```
  cat $sampleID |while read sample
  do
    $plink --allow-extra-chr --bfile $sample --homozyg --homozyg-window-het 3 --homozyg-window-missing 20 --out $sample.homology.he3.mis20
    Rscript ./src/ROH.v0.1.R --homfile $sample.homology.he3.mis20.hom -p $sample.bed -s $sample
   done
  ```
  - Calculation of distribution of ROH in different length categories and visualization
  ```
  
  ```

