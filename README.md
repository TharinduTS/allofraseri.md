# Mapping X. allofraseri RADSEQ data to the X. laevis genome

This project will map RADseq data that was generated using the SbfI restriction enzyme to the X. laevis genome version 9.1

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
