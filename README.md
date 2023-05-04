# PopGen_project
Population genomics GWAS project

# GWAS of eye color or height

In this project you will be looking at GWAS data from [openSNP](https://opensnp.org/), which is a web site where users of direct-to-customer genetic tests can share their personal data with other users. The phenotypes we will be looking at is self-reported eye color and height. When looking at the data you should be aware that:

- The data comes from different companies that use different chips so there are markers that are missing from some individuals because they were not present on the chip used by their company.
- The gender information is missing from the file and by default plink will ignore the phenotype of individuals without gender information. So you have to use “--allow-no-sex” option in plink.

## Investigate the following

A. Are there any closely related individuals in the sample?




B. Do a PCA plot. What does it tell you about the samples?

C. The files eye_color.txt and height.txt contains the self-reported eye color and height for the individuals in the data. Do a GWAS on one these traits. There are 12 eye color categories and you can group some of them together to create a binary phenotype. How many significant loci do you find?

D. Do additional analyses using plink, GCTA, R or any other tool you might find relevant. The list below contains some suggestions for possible analyses, but you can also come up with your ideas

Suggestions for further analysis:

- Use mixed model for GWAS.
- Do imputation (either of the whole genome or the region around the most significant SNP) and see if you can then find variants with lower p-values.
- If you use half of the data set to calculate a polygenic score, how well does that score predict height on the other half?
- Find a trained height PRS on the internet. How well does it predict the height in this data set?
- Test for epistasis.
- What are the distribution of phenotypes for each of the genotypes at the most significant SNP? If you want to analyse it in R you can use the "--recode A" together with the "--snp" and "--window" option in plink to get the variants around a specific SNP written to a text file that it is easy to load in R.
- How many of the significant variants found in the largest published GWAS study can you replicate those hits in this data set?
- Make association tests where you condition on most significant variant (you can use the --condition option in plink)



Stuff done:

## Removed individuals with elevated LD

```bash
plink --bfile gwas_data --indep 50 5 2 --allow-no-sex --out gwas_QC_LD
plink --bfile gwas_data --remove gwas_QC_LD.hh --make-bed --out gwas_QC
```

## Removed individuals with missing data or outlying heterozygosity

DO THIS: plink --bfile gwas_data --genome --out gwas_QC_IBD


```bash
plink --bfile gwas_data --missing --allow-no-sex --out gwas_QC
plink --bfile gwas_data --het --allow-no-sex --out gwas_QC


```

