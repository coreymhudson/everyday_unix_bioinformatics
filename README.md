everyday_unix_bioinformatics
============================
This repos documents the everyday unix functions that I use.

## **Using brace expansion to create directory structure**
``` shell
mkdir -p directory/{src,bin,lib,man}
```
After this is run, the directory structure will be as below
``` shell
$ find . | grep directory
  ./directory
  ./directory/bin
  ./directory/lib
  ./directory/man
  ./directory/src
```

## **Super simple gff to bed creator.**
This assumes the chromosome name is field 1 of the GFF file.
* Step 1: Pull fields 1, 4 and 5 (Name, Left and Right respectively)
* Step 2: Get rid of comment lines
* Step 3: Sort first by chromosome, then numerically by the left position
``` shell
cut -f1,4,5 Ncbi_Gff_File.gff | grep -v '^#' | sort -k1,1 -k2n,2
```

## **Super simple fastq to fasta in awk.**
``` shell
awk '{if(NR%4==1) {printf(">%s\n",substr($0,2));} else if(NR%4==2) print;}' file1.fastq > file1.fasta
```
