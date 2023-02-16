# UNIX Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`
```
$ du -h fang_et_al_genotypes.txt
$ wc -l fang_et_al_genotypes.txt
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
$ file fang_et_al_genotypes.txt
$ head fang_et_al_genotypes.txt | cut -f 1-20 | column -t
```

By inspecting this file I learned that:
1. The file size is 6.1M
2. The file contains 2783 lines, 986 columns, and is tab delimited
3. The file consists of only ASCII characters and has very long lines
4. The first line is a header that contains the SNP_ID starting in column 4
5. The genotype data starts in column 4
6. Column 3 contains the group needed to identify species

### Attributes of `snp_position.txt`
```
$ du -h snp_position.txt
$ wc -l snp_position.txt
$ awk -F "\t" '{print NF; exit}' snp_position.txt
$ file snp_position.txt
$ head snp_position.txt | column -t
$ cut -f 1 snp_position.txt | tail -n +2 | sort -c
```

By inspecting this file I learned that:
1. The file size is 38K
2. The file contains 984 lines, 15 columns, and is tab delimited
3. The file consists of only ASCII characters
4. The first line of the file is a header with column labels
5. SNP_ID is listed in column 1
6. column 1 is sorted

## Data Processing

### Maize Data

To extract maize data and SNP_IDs from `fang_et_al_genotypes.txt` and create `maize_genotypes.txt`:
```
$ head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
$ grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt >> maize_genotypes.txt
```

To transpose the genotype data:
```
$ awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```

To remove the unecessary header lines from `transposed_maize_genotypes` and sort by SNP_ID:
```
$  tail -n +4 transposed_maize_genotypes.txt | sort -k1,1 > transposed_maize_genotypes_sorted.txt
```

To remove header from `snp_position.txt`, cut out needed columns (SNP_ID, chromosome, position), and sort by SNP_ID:
```
$ tail -n +2 snp_position.txt | cut -f 1,3,4 | sort -k1,1 > SNP_ID_chrm_position_sorted.txt
``` 

To join `SNP_ID_chrm_position_sorted.txt` and `transposed_maize_genotypes_sorted.txt`:
```
$ join -1 1 -2 1 -t $'\t' SNP_ID_chrm_position_sorted.txt transposed_maize_genotypes_sorted.txt > maize_SNP_chrm_pos.txt
```

To extract records for SNPs with unknown position in genome:
```
$ awk '$2 == "unknown" || $3 == "unknown"' maize_SNP_chrm_pos.txt > maize_unknown.txt
```

To extract records for SNPs with multiple positions in the genome:
```
$ awk '$2 == "multiple" || $3 == "multiple"' maize_SNP_chrm_pos.txt > maize_multiple.txt
```

To extract records for each chromosome into their own files:
```
$ for i in {1..10}; do awk -v i="$i" '$2 == i && $3 != "multiple" && $3 != "unknown"' maize_SNP_chrm_pos.txt > maize_chr_$i.txt; done
```

To sort the individual chromosome files by increasing position value:
```
$ for i in {1..10}; do sort -k3,3 -n maize_chr_$i.txt > maize_incr_chr_$i.txt; done
```

To sort the individual chromosome files by decreasing postition value and replace "?" with "-" (missing data):
```
$ for i in {1..10}; do sort -k3,3 -nr maize_chr_$i.txt | sed 's/?/-/g' > maize_decr_chr_$i.txt; done
```
 
### Teosinte Data

To extract teosinte data and SNP_IDs from `fang_et_al_genotypes.txt` and create `teosinte_genotypes.txt`:
```
$ head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
$ grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```

To transpose the genotype data:
```
$ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```

To remove the unecessary header lines from `transposed_maize_genotypes` and sort by SNP_ID:
```
$  tail -n +4 transposed_teosinte_genotypes.txt | sort -k1,1 > transposed_teosinte_genotypes_sorted.txt
```

To join `SNP_ID_chrm_position_sorted.txt` and `transposed_maize_genotypes_sorted.txt`:
```
$ join -1 1 -2 1 -t $'\t' SNP_ID_chrm_position_sorted.txt transposed_teosinte_genotypes_sorted.txt > teosinte_SNP_chrm_pos.txt
```

To extract records for SNPs with unknown position in genome:
```
$ awk '$2 == "unknown" || $3 == "unknown"' teosinte_SNP_chrm_pos.txt > teosinte_unknown.txt
```

To extract records for SNPs with multiple positions in the genome:
```
$ awk '$2 == "multiple" || $3 == "multiple"' teosinte_SNP_chrm_pos.txt > teosinte_multiple.txt
```

To extract records for each chromosome into their own files:
```
for i in {1..10}; do awk -v i="$i" '$2 == i && $3 != "multiple" && $3 != "unknown"' teosinte_SNP_chrm_pos.txt > teosinte_chr_$i.txt; done
```

To sort the individual chromosome files by increasing position value:
```
$ for i in {1..10}; do sort -k3,3 -n teosinte_chr_$i.txt > teosinte_incr_chr_$i.txt; done
```

To sort the individual chromosome files by decreasing postition value and replace "?" with "-" (missing data):
```
$ for i in {1..10}; do sort -k3,3 -nr teosinte_chr_$i.txt | sed 's/?/-/g' > teosinte_decr_chr_$i.txt; done
```




