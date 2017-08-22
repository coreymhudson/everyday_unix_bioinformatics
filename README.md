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

### **Simple command to compare the sizes of two files.**
``` shell
bc -l <<< `stat -c "%s" file1`/`stat -c "%s" file2`
```

### **Easiest bz2 to gzip conversion.**
``` shell
bunzip2 -c file.bz2 | gzip > file.gz
```

### **Report only the highest scoring hmmsearch tblout results.**
``` shell
grep -v "^#" results.tbl | awk '{print $1"\t"$4"\t"$5}' | sort -k1,1 -k3g | awk '$1!=h {print} {h=$1}'
```

### **Break fasta at first space.**
``` shell
awk '{if ($1 ~ />/){print $1} else {print $0}}' file1.fasta > file2.fasta
```

### **Find the average sequence length in a fastq file.**
```
awk '(NR%4==2){l+=length; i+=1} END {print l/i}' file.fastq
```

### **Convert fasta to CSV...you know, as one always does.***
```
echo '"name","sequence"' && cat file.fna | cut -f1 -d" " | awk '{if ($1 ~ ">") {if(h != ""){print "\""h"\",\""s"\""}; s="";h=substr($1,2);} else{s=s$1}} END {print "\""h"\",\""s"\""}'
```

### **Find all the .fna files added today and gzip them.**
```
find ./*.fna -mtime -1 -type f -exec gzip {} \;
```

### **Alternatively the last task can be done in parallel. Major time savings (sometimes).**
```
find ./*.fna -mtime -1 -type f -print0 | parallel -q0 gzip
```

### **Find repeat regions in the same genome, using blast.**
Set up the database
```
makeblastdb -in sequence.fa -out sequence -dbtype nucl
```
Query the database for all self-hits that aren't that aren't the same region
```
blastn -query sequence.fa -db sequence -outfmt 6 | awk '($1==$2)&&(($7!=$9)||($8!=$10)){print $0}'
```
If the sequence.fa file is large, you can gain serious speedups by parallelizing
```
cat sequence.fa | parallel --block 50k --recstart '>' --pipe blastn -outfmt 6 -db sequence -query - | awk '($1==$2)&&(($7!=$9)||($8!=$10)){print $0}'
```

### **Grep a list of hmms from an hmm database (this may be too simple).**
```
hmmfetch -f list_of_pfams Pfam-A.hmm > shortened_pfams.hmm
```

### **Find all files larger than 5gb and sort the output by size.**
```
find . -type f -size +5G -exec ls -lShr {} \;
```

### **Gzip all fq files in a directory.**
```
parallel --gnu gzip  ::: *.fq
```

### **Concatenate a file into an already compressed file.**
``` 
gzip -c file.fq >> compress_file.fq.gz
```

### **List only the files in the current directory that are currently being written.**
```
lsof | grep `pwd` | grep '1w' | awk '{print $9}' | xargs -r ls -l
```

### **A job is running...I want to start a new job as soon as it stops.**
This will print "Waiting..." to the screen until the job is done and everything is ready.
```
echo Waiting...; while ps -p $PID > /dev/null; do sleep 1; done; nohup script.sh &
```

### **Delete all files in directory but one.**
```
find . -type f -not -name "NC_000913.fna" | xargs rm
```

### **Parallellize prodigal.**
```
cat output.fna | parallel --block 50k --recstart '>' --pipe ~/Desktop/Job/Sandia/software/Prodigal/prodigal -a /dev/stdout -d /dev/stderr -q -p meta -o output_test.gff -f gff 1>>out.faa 2>>out.ffn
```
