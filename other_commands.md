Difference
```
vcftools --gzvcf M3_q20_dp5_calls.vcf.gz --gzdiff M4_q20_dp5_calls.vcf.gz --diff-site --out in3_v_in4_q20_dp5

```
To filter and output specific columns
```
bcftools query -i'DP>=5' -f'%POS %REF %ALT %QUAL %DP\n' M4_q20_dp5_calls.norm.flt-indels.bcf

bcftools query -i'QUAL>20 && DP>5' -f'%POS %REF %ALT %QUAL %DP\n' M4_calls.vcf.gz
```
To put the filename as the fasta header
```
awk '/^>/ {gsub(/.fa(sta)?$/,"",FILENAME);printf(">%s\n",FILENAME);next;} {print}' M4_q20_dp5_consensus.fa > M4_q20_dp5_consensus.fa
```
Coverage statistics
```
samtools coverage M1_sort_rmdup.bam
```
