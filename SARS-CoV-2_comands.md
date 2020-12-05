# SARS-CoV-2 commands
QC raw reads
```
java -jar /Users/kalilamali/Trimmomatic-0.39/trimmomatic-0.39.jar PE -phred33 M4_R1.fastq M4_R2.fastq M4_R1_paired.fastq M4_R1_unpaired.fastq M4_R2_paired.fastq M4_R2_unpaired.fastq ILLUMINACLIP:/Users/kalilamali/Trimmomatic-0.39/adapters/TruSeq3-PE.fa:2:30:10:2:keepBothReads LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:40
```
De-novo assembly
```
megahit -1 M4_R1_paired.fastq -2 M4_R2_paired.fastq -o M4_denovo.fa
```
Map reads to reference genome
```
bwa index NC_045512.fasta

bwa mem NC_045512.fasta M4_R1_paired.fastq M4_R2_paired.fastq > M4.sam

samtools view -b M4.sam > M4.bam

samtools sort M4.bam -o M4_sort.bam
```
Mark and remove duplicates
```
picard MarkDuplicates -I M4_sort.bam -O M4_sort_rmdup.bam -M M4_sort_rmdup_metrics.txt --REMOVE_DUPLICATES
```
Variant calling
```
picard AddOrReplaceReadGroups -I M4_sort_rmdup.bam -O M4_sort_rmdup_group.bam -SORT_ORDER coordinate -RGID foo -RGLB bar -RGPL illumina -RGPU NONE -RGSM M4 -CREATE_INDEX True

samtools index M4_sort_rmdup_group.bam

bcftools mpileup -q 25 -Q 35 -Ou -f NC_045512.fasta M4_sort_rmdup_group.bam | bcftools call -mv -Oz -o M4_calls.vcf.gz

bcftools view -i 'QUAL>=20 && DP>=5' M4_calls.vcf.gz | bgzip -c > M4_q20_dp5_calls.vcf.gz
```
Consensus
```
bcftools norm -f NC_045512.fasta M4_q20_dp5_calls.vcf.gz -Ob -o M4_q20_dp5_calls.norm.bcf

bcftools filter --IndelGap 5 M4_q20_dp5_calls.norm.bcf -Ob -o M4_q20_dp5_calls.norm.flt-indels.bcf

bcftools index M4_q20_dp5_calls.norm.flt-indels.bcf

cat  NC_045512.fasta | bcftools consensus M4_q20_dp5_calls.norm.flt-indels.bcf > M4_q20_dp5_consensus.fa
```
