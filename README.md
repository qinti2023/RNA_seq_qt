# Pepline of RNA seq [upstream and downstream]

**Designed and created by qinti**

> Reference : Ruobing Li, **Ti Qin**, Yabo Guo, Shan Zhang, Xiaogang Guo,
> CEAM is a mitochondrial-localized, amyloid-like motif-containing microprotein expressed in human cardiomyocytes,
> Biochemical and Biophysical Research Communications,
> Volume 734,2024,150737,ISSN 0006-291X,https://doi.org/10.1016/j.bbrc.2024.150737.

### Upstream[linux]

- Qc fastq files in batch---Fastqc
- Data cleaning---Trim-galore
- Mapping to genome---Hisat2 and Samtools
- Getting  counts according to GTF ---featureCounts

### Downstream[R]

- Step 0 : packages library
- Step 1 : Data input and clean
- Step 2 : Different experssion analysis
- Step 3 : Visualization
- Step 4 : KEGG and GO enrichment
- Step 5 : GSEA---Gene Set Enrichment Analysis
- Step 6 : heatmap multiple groups