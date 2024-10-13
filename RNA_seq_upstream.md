# RNA_seq pepline (upstream)

## Environment and Software

```shell
conda create -n RNA_seq python=3.9
conda activate RNA_seq
mamba install -y fastqc multiqc hisat2 samtools sybread trim-galore
```

## Qc fastq files in batch---Fastqc

```shell
cd ./rawdata
mkdir ../QCresult
nohup fastqc -t 6 -o ../QCresult *.fq >qc.log &
cd ../QCresult
multiqc *.zip
```

## Data cleaning---Trim-galore

```shell
cd ~/RNA_seq/CleanDataSet
### list ids of all samples
ls ~/RNA_seq/rawdata/ *_1.fq | awk -F'/' '{print $NF}' | cut -d'_' -f1 >ID
### create cleaning script 
touch trim_galore.sh
vim trim_galore.sh
rawdata=~/RNA_seq/rawdata/
cleandata=~/RNA_seq/CleanDataSet
cat ID | while read id
do
  trim_galore -q 20 --length 36 --max_n 3 --stringency 3 --fastqc --paired -o ${cleandata} ${rawdata}/${id}_1.fq ${rawdata}/${id}_2.fq
done
### submit the task to the bachground
nohup sh trim_galore.sh >trim_galore.log &
### qc fastq files after adapter-removing
multiqc *.zip
```

## Mapping to genome---Hisat2 and Samtools

```shell
### download the index in hisat2 website
mkdir ~/RNA_seq/Reference
cd ~/RNA_seq/Reference
wget https://genome-idx.s3.amazonaws.com/hisat/mm10_genome.tar.gz
tar -zxvf ./mm10_genome.tar.gz 
rm ./mm10_genome.tar.gz 
### creat mapping script
mkdir ~/RNA_seq/Mapping
cd ~/RNA_seq/Mapping
touch Hisat.sh
vim Hisat.sh
index=~/RNA_seq/Reference/mm10/genome
inputdir=~/RNA_seq/CleanDataSet/
outdir=~/RNA_seq/Mapping
cat ~/RNA_seq/CleanDataSet/trim_galore/ID | while read id
do
  hisat2 -p 20 -x ${index} -1 ${inputdir}/${id}_1_val_1.fq -2 ${inputdir}/${id}_2_val_2.fq 2>${id}.log | samtools sort -@ 10 -o ${outdir}/${id}.Hisat_aln.sorted.bam - && samtools index ${outdir}/${id}.Hisat_aln.sorted.bam ${outdir}/${id}.Hisat_aln.sorted.bam.bai
done
### submit the task to background
nohup sh Hisat.sh >Hisat.log &
### summary the mapping result
multiqc -o ./ *log
```

## Getting  counts according to GTF ---featureCounts

```shell
### downloading the annotation file
cd ~/RNA_seq/Reference
wget -b https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M25/gencode.vM25.primary_assembly.annotation.gtf.gz
### creat quantifing script
mkdir ~/RNA_seq/Counts
cd ~/RNA_seq/Counts
vim featureCounts.sh
gtf=~/RNA_seq/Reference/gencode.vM25.primary_assembly.annotation.gtf.gz
inputdir=~/RNA_seq/Mapping
featureCounts -T 6 -p -t exon -g gene_id -a $gtf -o all.id1.txt $inputdir/*.sorted.bam
### Quantification 
sh featureCounts.sh
### summary the quantitative results
multiqc all.id.txt.summary
#### counts matrix 
cat all.id.txt | cut -f1,7- > counts.txt
less -S all.id1.txt |grep -v '#' |cut -f 1,7- |sed 's#/home/lsq/RNA_seq/Mapping//##g' |sed 's#.Hisat_aln.sorted.bam##g' >raw_counts1.txt
```



