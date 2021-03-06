# Mapping X. allofraseri RADSEQ data to the X. laevis genome

This project will map RADseq data that was generated using the SbfI restriction enzyme to the X. laevis genome version 9.1

# Files are in the directory

/2/scratch/tharindu/allofraseri

# QC and Trimming 
The first step is to assess quality using fastqc and also identify repetitive sequences in the data (also using fastqc).  This step was already done previously by Ben Furman after demultiplexing the data.  I worked with BenE to make a perl script that does the trimming using `Trimmomatic` version 0.36 and using a modified adapter file in which we added repetitive sequences that were identified with `fastqc` to an adapter file that comes with Trimmomatic ('TruSeq2-PE_for_allofraseri.fa').

```perl

#!/usr/bin/perl
# This script will use trimmomatic to trim all of the fastq reads  

my $status;
my @files;

@files = glob("./Cam*.fq.gz");

foreach(@files){
    my $commandline = "java -jar  /usr/local/trimmomatic/trimmomatic-0.36.jar SE -phred33 -trimlog ".$_."_log.txt ".$_." ".$_."_trimmed.fq.gz ILLUMINACLIP:./TruSeq2-PE_for_allofraseri.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36";
    print $commandline,"\n";
    $status = system($commandline);
}

```
make this executable by chmod -x or use
```
perl <script_name>
```
# Combining duplicate sequences

```bash
cat CamMale4_a.fq.gz_trimmed.fq.gz CamMale4_b.fq.gz_trimmed.fq.gz >CamMale4_a.fq.gz_trimmed.fq.gz_c
```

(and then renamed CamMale4_a.fq.gz_trimmed.fq.gz_c as CamMale4_a.fq.gz_trimmed.fq.gz to match other files)

# FastQC

```
fastqc <file_name>
```
(available version was fastqc_v0.11.3)

# Preparing reference genome

Downloaded X. laevis 9.1 genome using
```
wget http://ftp.xenbase.org/pub/Genomics/JGI/Xenla9.1/Xla.v91.repeatMasked.fa.gz
```
Unzip the genome
```
gunzip Xla.v91.repeatMasked.fa.gz
```
and then index it
```
bwa index -a bwtsw Xla.v91.repeatMasked.fa

samtools faidx Xla.v91.repeatMasked.fa
```
and now make a dict file:
```
java -jar /usr/local/picard-tools/picard.jar CreateSequenceDictionary REFERENCE=Xla.v91.repeatMasked.fa OUTPUT=Xla.v91.repeatMasked.dict

```
#  OPTIONAL**************************************************
(to help creating the files that can be easily handled with following script)
# Abstracting parts of file names
```
ls | grep trimmed.fq.gz_trimmed.fq | sed 's/_a.fq.gz_trimmed.fq.gz_trimmed.fq/.gz/g'
```
# removing .gz

```
ls | grep Cam | sed 's/.gz//g'
```
# changing the names of multiple files
```
for filename in *; do newname=`echo $filename | sed 's/_a.fq.gz_trimmed.fq.gz_trimmed.fq/.gz/g'`; mv $filename $newname; done
```
(used "*" because all the files in the directory needed to be renamed)

# *************************************************************

# Align the fq files using bwa mem and sort them too
```
path_to_data="final_renamed"
path_to_chromosome="reference_genome"
chromosome="Xla.v91.repeatMasked"

individuals="CamFemale1
CamFemale2
CamFemale3
CamFemale4
CamMale1
CamMale2
CamMale3
CamMale4
CamMale5"

for each_individual in $individuals
do

echo ${each_individual}
    bwa mem -M -t 16 -r "@RG\tID:FLOWCELL1.LANE6\tSM:${each_individual}\tPL:illumina" $path_to_chromosome/$chromosome.fa $path_to_data/${each_individual}.fq | samtools view -bSh - > $path_to_data/${each_individual}.bam
    samtools sort $path_to_data/${each_individual}.bam -o $path_to_data/${each_individual}_sorted.bam
    samtools index $path_to_data/${each_individual}_sorted.bam
done
```
# opening a new screen
```
screen -S <screen_name>
```
send it to background

Ctrl+a+d

Re-open with
```
screen -r <screen_name>
```
# Merging bam files, filtering and creating vcf
```
samtools mpileup -d8000 -ugf ../reference_genome/Xla.v91.repeatMasked.fa -t DP,AD CamFemale1_sorted.bam  CamFemale4_sorted.bam  CamMale3_sorted.bam CamFemale2_sorted.bam  CamMale1_sorted.bam    CamMale4_sorted.bam CamFemale3_sorted.bam  CamMale2_sorted.bam    CamMale5_sorted.bam | bcftools call -V indels --format-fields GQ -m -O z | bcftools filter -e 'FORMAT/GT = "." || FORMAT/DP < 10 || FORMAT/GQ < 20 || FORMAT/GQ = "."' -O z -o Cam_merged_sorted.bam.vcf.gz
```
# Command above had filtered many sites and had no data left to work with. Therefore had to go with the following command, reducing filtering steps
```
samtools mpileup -d8000 -ugf ../reference_genome/Xla.v91.repeatMasked.fa -t DP,AD CamFemale1_sorted.bam  CamFemale4_sorted.bam  CamMale3_sorted.bam CamFemale2_sorted.bam  CamMale1_sorted.bam    CamMale4_sorted.bam CamFemale3_sorted.bam  CamMale2_sorted.bam    CamMale5_sorted.bam | bcftools call -V indels --format-fields GQ -m -O z -O z -o Cam_merged_sorted.bam.vcf.gz
```
# For Trop, Laevis,Gilli filtered VCF again with

```bash
vcftools --gzvcf ../all_sorted_bam/xlaevis_and_xgilli_sorted.bam.vcf.gz --minGQ 20 --minDP 25 --recode --recode-INFO-all --out ../final_vcf/xlaevis_and_xgilli_all_final
```

