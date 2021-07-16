
# Running a GWAS workflow on BioData Catalyst

On the BioData Catalyst platform, locate the following public apps and copy them to your project. To find the apps, go to Public Gallery > Apps, and then use the search box. Click "copy", select your project from the dropdown, and click "copy" again.

1. Bcftools Merge and Filter
2. VCF to GDS Converter
3. KING robust
4. PC-AiR
5. PC-Relate
6. GENESIS Null Model
7. GENESIS Single Variant Association Testing
8. GENESIS Aggregate Association Testing

## Preparing the genotype data

### Bcftools Merge and Filter

Run the `Bcftools Merge and Filter` tool to combine two separate VCF files into one merged file (see the task "1. Bcftools merge chr 1 subsets"). 

- Inputs
  - Input variant files: `1KG_phase3_chr1.subset1.vcf` and `1KG_phase3_chr1.subset2.vcf`
- App Settings
  - Output name: `1KG_phase3_subset_chr1`
  
This will create a file named `1KG_phase3_subset_chr1.merged.vcf.gz` that contains all of the data from both input files. 

### VCF to GDS Converter

Run the `VCF to GDS Converter` workflow to convert the 1000 Genomes VCF file you just created to a GDS file (see the task "2. Convert chr 1 VCF to GDS"). 

- Inputs
  - Variants Files: `1KG_phase3_subset_chr1.merged.vcf.gz` 
- App Settings
  - check GDS: No
  
This will create a GDS file named `1KG_phase3_subset_chr1.merged.gds` that contains the same information as the input VCF file. Use the "view stats and logs" button to check on the status of your tasks.

## Ancestry and Relatedness

### KING robust

Run the `KING robust` workflow to compute initial kinship estimates for all pairs of samples (see the task "3. KING robust run").

- Inputs
  - GDS file: `1KG_phase3_subset_chr1.merged.gds`
- App Settings
  - Output prefix: `1KG_phase3_subset_chr1`
  
This will create a `1KG_phase3_subset_chr1_king_robust.gds` file that contains the KING estimates as well as a `1KG_phase3_subset_chr1_king_robust_all.pdf` with the plot of estimated kinship vs. IBS0. 

Use the "view stats and logs" button to check on the status of your tasks. Click on the bar that says "king_robust", click "view logs", and click the "king_robust" folder. In here you can see detailed logs of the job; take a look at the `job.out.log` and `job.err.log` -- these can be useful for debugging issues. 

### PC-AiR

Run the `PC-AiR` workflow to compute ancestry principal components (see the task "4. PC-AiR run").

- Inputs
  - Full GDS File: `1KG_phase3_subset_chr1.merged.gds`
  - Kinship File: `1KG_phase3_subset_chr1_king_robust.gds`
  - Phenotype File: `sample_phenotype_pcs.RData`
  - Pruned GDS File: `1KG_phase3_subset_chr1.merged.gds`
- App Settings
  - Output prefix: `1KG_phase3_subset_chr1`
  - Group: "Population"
  - Run PC-variant correlation: FALSE
  
This will create a `1KG_phase3_subset_chr1_related.RData` file that contains the output PCs as well as several PC plots. Take a look at the `1KG_phase3_subset_chr1_pca_parcoord.pdf` -- how many PCs to adjust for in PC-Relate and the association analyses?

### PC-Relate

Run the `PC-Relate` workflow to compute ancestry adjusted relatedness estimates (see the task "5. PC-Relate run").

- Inputs
  - GDS File: `1KG_phase3_subset_chr1.merged.gds`
  - PCA File: `1KG_phase3_subset_chr1_related.RData`
- App Settings
  - Number of PCs: 2
  - Output prefix: `1KG_phase3_subset_chr1`
  
This will create a `1KG_phase3_subset_chr1_pcrelate.RData` file that contains the PC-Relate relatedness estimates as well as a `1KG_phase3_subset_chr1_pcrelate_all.pdf` with the plot of estimated kinship vs. k0. (Note that the estimates are very noisy because we are using so few variants in this toy example.)

From the "view stats and logs" page, click "instance metrics" to see an overview of CPU, RAM, etc. usage; this will update as the process runs. 

## Association Testing

### Null Model

Fit a null model using the `Null Model` workflow (see the task "6. GENESIS Null Model run"). 

- Inputs
  - Phenotype File: `sample_phenotype_pcs.RData` (note that the PCs are included in this file)
  - Relatedness matrix file: `kinship.RData`
- App Settings
  - Outcome: height
  - Covariates: age, sex, study, PC1, PC2, PC3, PC4
  - Group variate: study
  - Family: gaussian
  - Output prefix: `height`

This will create a `height_null_model.RData` file that contains the null model fit and a `height_phenotypes.RData` file that contains the phenotype data used in the analysis. It also creates a `height_report.html` null model report -- review this report.

### Single variant association test

Use the `GENESIS Single Variant Association Testing` workflow to run a single variant association test (see the task "7. GENESIS Single Variant Association Testing run").

- Inputs
  - GDS files: `1KG_phase3_subset_chr1.merged.gds`
  - Null model file: `height_null_model.RData`
  - Phenotype file: `height_phenotypes.RData` (it is recommended you use the file produced by the Null Model workflow)
- App Settings
  - MAC threshold: 5
  - Output prefix: `height`
  - memory GB: 64

Review the QQ and manhattan plots.


### Aggregate variant test

Use the `GENESIS Aggregate Association Testing` workflow to run a burden association test (see the task "8. GENESIS Aggregate Association Testing run").

- Inputs
  - GDS files: `1KG_phase3_subset_chr1.merged.gds`
  - Null model file: `height_null_model.RData`
  - Phenotype file: `height_phenotypes.RData` (it is recommended you use the file produced by the Null Model workflow)
  - Variant group files: `variants_by_gene_chr1.RData`
- App Settings
  - Alt Freq Max: 0.1
  - Test: burden
  - Output prefix: `height`

Review the QQ and manhattan plots.


## Analysis follow-up

In RStudio, locate the results of your association test under `/sbgenomics/project-files/`. Load one of these results files into R and explore it.

