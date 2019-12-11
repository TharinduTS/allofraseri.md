# Mapping X. allofraseri RADSEQ data to the X. laevis genome

This project will map RADseq data that was generated using the SbfI restriction enzyme to the X. laevis genome version 9.1

# QC and Trimming 
The first step is to assess quality using fastqc and also identify repetitive sequences in the data (also using fastqc).  This step was already done previously by Ben Furman after demultiplexing the data.  I worked with BenE to make a perl script that does the trimming using `Trimmomatic` version 0.32 and using a modified adapter file in which we added repetitive sequences that were identified with `fastqc` to an adapter file that comes with Trimmomatic (put name here).

```perl
copy and paste the script here
```
