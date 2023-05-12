# PopGen_project
Population genomics GWAS project

# GWAS of eye color or height

In this project you will be looking at GWAS data from [openSNP](https://opensnp.org/), which is a web site where users of direct-to-customer genetic tests can share their personal data with other users. The phenotypes we will be looking at is self-reported eye color and height. When looking at the data you should be aware that:

- The data comes from different companies that use different chips so there are markers that are missing from some individuals because they were not present on the chip used by their company.
- The gender information is missing from the file and by default plink will ignore the phenotype of individuals without gender information. So you have to use “--allow-no-sex” option in plink.

## Investigate the following

A. Are there any closely related individuals in the sample?
319 identified individuals have been removed due to close relatedness

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

## Removed individuals with missing data or outlying heterozygosity
```bash
plink --bfile gwas_data --missing --out GWAS_QC
```

```r  
d_miss <- read.table("gwas_QC.imiss", header = T)
d_het <- read.table("gwas_QC.het", header = T)
d <- inner_join(d_miss, d_het)
d <- left_join(d, metadata, by = "IID")

d %>% 
  mutate(O.HET = (N.NM. - O.HOM.) / N.NM.) -> d
  
d %>% 
  filter(F_MISS >= 0.95 | 
         O.HET > mean(O.HET) + 3 * sd(O.HET) |
         O.HET < mean(O.HET) - 3 * sd(O.HET)) %>% 
  select(FID, IID) -> wrong_missing
write.table(wrong_missing, file = "wrong_missing.txt", col.names = F, row.names = F)
```

```bash
plink --bfile gwas_data --remove wrong_missing.txt --allow-no-sex --make-bed --out GWAS_QC_rm
```

## Identity by descent 

```bash
plink --bfile GWAS_QC_rm --genome --min 0.185 --out GWAS_QC_IBD
```

```r
ibd <- read.table('GWAS_QC_IBD.genome', header = TRUE)
members <- ibd$FID1
members <- unique(members)
write.table(cbind(members,members), file = 'wrong_ibd.txt', col.names = F, row.names = F)
```

```bash
plink --bfile GWAS_QC_rm --remove wrong_ibd.txt --make-bed --out GWAS_QC_rm_ibd
```

## PCA

```bash
plink --bfile GWAS_QC_rm_ibd --pca 20 --out GWAS_pca
```

```r
pca <- read.table("GWAS_pca.eigenvec")

ggplot(data = pca, aes(x = V3, y = V4)) +
  geom_point()

pca %>% 
  filter(V4 < - .10) -> pca_outliers

write.table(cbind(pca_outliers$V1, pca_outliers$V2), file = "pca_outliers.txt", col.names = F, row.names = F)
```

```bash
plink --bfile GWAS_QC_rm_ibd --remove pca_outliers.txt --make-bed --out GWAS_QC_rm_ibd_pca
```

```r
pca <- read.table("GWAS_pca_rm-outliers.eigenvec")

ggplot(data = pca, aes(x = V3, y = V4)) +
  geom_point()
  
# Eigenvalues
ggplot(data = eigenvals) +
  geom_col(aes(x = 1:20, y = V1)) +
  labs(x = "PC",
       y = "Explained variance") +
  scale_x_continuous(breaks = 1:20) +
  theme_pander()

# Cumulative sum of eigenvalues
ggplot(data = eigenvals) +
  geom_point(aes(x = 1:20, y = cumsum(V1))) +
  labs(x = "PC",
       y = "Cumulative sum") + 
  scale_x_continuous(breaks = 1:20) +
  scale_y_continuous(limits = c(0, 100)) +
  theme_pander()
```




