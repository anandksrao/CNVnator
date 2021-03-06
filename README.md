# README 

## Quick start guide

```
# Extract read mapping
$ ./cnvnator -root file.root -tree file.bam -unique

# Generate histogram
$ ./cnvnator -root file.root -his 1000 -chrom 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 X Y
  OR
$ ./cnvnator -root file.root -his 1000 -chrom chr1 chr2 chr3 chr4 chr5 chr6 chr7 chr8 chr9 chr10 chr11 chr12 chr13 chr14 chr15 chr16 chr17 chr18 chr19 chr20 chr21 chr22 chrX chrY

# Calculate statistics
$ ./cnvnator -root file.root -stat 1000 -d dir_with_genome_fa/

# Partition
$ ./cnvnator -root file.root -partition 1000

# Call CNVs
$ ./cnvnator -root file.root -call 1000
```

## 1. Compilation

### Dependencies

You must install [ROOT package](http://root.cern.ch) and set up `$ROOTSYS` variable (see ROOT documentation [here](https://root.cern.ch/root/html534/guides/users-guide/GettingStarted.html)).

Also, a link to the samtools binary should be present in your CNVnator directory.

If compilation is not completed but the file libbam.a has been created, you can continue.

### Installation from github

```
git clone https://github.com/abyzovlab/CNVnator.git

cd CNVnator

ln -s /path/to/src/samtools samtools

make
```

If make doesn't work, try `"make OMP=no"` which will disable parallel support.

### Installing with Yeppp support

[Yeppp](http://www.yeppp.info/) is a library which provides high-performance implementations of math functions.

To install with Yeppp support, download Yeppp from [here](http://bitbucket.org/MDukhan/yeppp/downloads/yeppp-1.0.0.tar.bz2)
and extract it to a location of your choice. Set `YEPPPLIBDIR` and `YEPPPINCLUDEDIR` directories appropriately. 

Typically, for Linux-based systems on x86-64, `YEPPPLIBDIR` will be yeppp-1.0.0/binaries/linux/x86_64/ and `YEPPPINCLUDEDIR` will be
yeppp-1.0.0/library/headers. 

To build, type  
`make YEPPPLIBDIR=... YEPPPINCLUDEDIR=...`  

To disable OpenMP, add `OMP=no` to the make command.

## 2. Predicting CNV regions

Running CNVnator involves a few steps outlined below. Chromosome names and lengths are
parsed from the input sam/bam file header. 

### 2.1 EXTRACTING READ MAPPING FROM BAM/SAM FILES

```
$ ./cnvnator -root out.root [-chrom name1 ...] -tree [file1.bam ...]
```
where,

out.root  -- output ROOT file. See ROOT package documentation.  
name1 ... -- chromosome name(s).  
file1.bam ...  -- bam file(s).  

Chromosome names must be specified the same way as they are described in the sam/bam
header, e.g., chrX or X. One can specify multiple chromosomes separated by
space. If no chromosome is specified, read mapping is extracted for all chromosomes
in the sam/bam file. Note that this would require machines with a large physical
memory of at least 7Gb. Extracting read mapping for subsets of chromosomes is a way
around this issue. Also note that the root file is not being overwritten.


Example:

```
./cnvnator -root NA12878.root -chrom 1 2 3  -tree NA12878_ali.bam
```

for bam files with a header like this:  
@HD VN:1.4    GO:none  SO:coordinate  
@SQ SN:1      LN:249250621  
@SQ SN:2      LN:243199373  
@SQ SN:3      LN:198022430  
...  

or

```
./cnvnator -root NA12878.root -chrom chr1 chr2 chr3 -tree NA12878_ali.bam
```
for bam files with a header like this:  
@HD VN:1.4    GO:none  SO:coordinate  
@SQ SN:chr1   LN:249250621  
@SQ SN:chr2   LN:243199373  
@SQ SN:chr3   LN:198022430  
...  

Example:

```
./cnvnator -root NA12878.root -chrom 4 5 6 -tree NA12878_ali.bam
./cnvnator -root NA12878.root -chrom 7 8 9 -tree NA12878_ali.bam
```

is equivalent to

```
./cnvnator -root NA12878.root -chrom 4 5 6 7 8 9 -tree NA12878_ali.bam
```


### 2.2 GENERATING A HISTOGRAM

```
$ ./cnvnator -root file.root [-chrom name1 ...] -his bin_size [-d dir]
```

This step is not memory consuming and so can be done for all chromosomes at once. 
It can also be carried for a subset of chromosomes. 
Files with individual chromosome sequences (.fa) are required and should reside in the 
current directory or in the directory specified by the -d option. 
Files should be named as: chr1.fa, chr2.fa, etc.


### 2.3 CALCULATING STATISTICS

```
$ ./cnvnator -root file.root [-chrom name1 ...] -stat bin_size
```

This step must be completed before proceeding to partitioning and CNV calling.


### 2.4 RD SIGNAL PARTITIONING

```
$ ./cnvnator -root file.root [-chrom name1 ...] -partition bin_size [-ngc]
```

Option `-ngc` specifies not to use GC corrected RD signal. Partitioning is the most time consuming step.

### 2.5 CNV CALLING

```
$ ./cnvnator -root file.root [-chrom name1 ...] -call bin_size [-ngc]
```

Calls are printed to STDOUT by default. You may redirect them to a file using the redirect operator >

The output columns are as follows:

CNV\_type coordinates CNV\_size normalized\_RD e-val1 e-val2 e-val3 e-val4 q0

where,

normalized_RD -- read depth normalized to 1.  
e-val1        -- is calculated using t-test statistics.  
e-val2        -- is from the probability of RD values within the region to be in
the tails of a gaussian distribution describing frequencies of RD values in bins.  
e-val3        -- same as e-val1 but for the middle of CNV  
e-val4        -- same as e-val2 but for the middle of CNV  
q0            -- fraction of reads mapped with q0 quality  

### 2.6 MERGING ROOT FILES

```
./cnvnator -root out.root [-chrom name1 ...] -merge file1.root ...
```
Merging can be used when combining read mappings extracted from multiple files.  
Note: histogram generation, statistics calculation, signal partitioning, and
CNV calling should be completed/redone after merging.


### 2.7 VISUALIZING SPECIFIED REGIONS

```
./cnvnator -root file.root [-chrom name1 ...] -view bin_size [-ngc]
```

Once prompted, enter a genomic region, e.g.,

```
>12:11396601-11436500
 or
>chr12:11396601-11436500
 or 
>12 11396601 11436500
 or
>chr12 11396601 11436500
```

Additionally, one can specify the length of flanking regions (default is 10 kb) to
be displayed as well, e.g.,

```
>12:11396601-11436500 100000
 or
>chr12:11396601-11436500 100000
 or
>12 11396601 11436500 100000
 or
>chr12 11396601 11436500 100000
```

One can also perform instant genotyping by adding the word 'genotype', e.g.,

```
>12:11396601-11436500 genotype
 or
>chr12:11396601-11436500 genotype
 or
>12 11396601 11436500 genotype
 or
>chr12 11396601 11436500 genotype
```

## 3. Genotyping genomic regions

For efficient genotype calculations, we recommend that you sort the list of regions by
chromosomes.

```
./cnvnator -root file.root -genotype bin_size [-ngc]
```

Once prompted enter a genomic region, e.g., 

```
>12:11396601-11436500
 or
>chr12:11396601-11436500
 or 
>12 11396601 11436500
 or
>chr12 11396601 11436500
```

One can also perform instant visualization by adding the word 'view', e.g.,

```
>12:11396601-11436500 view
 or
>chr12:11396601-11436500 view
 or
>12 11396601 11436500 view
 or
>chr12 11396601 11436500 view
```

### Additional notes

For genotyping of multiple regions one can use input piping, e.g.,
```
./cnvnator -root NA12878.root -genotype 100 << EOF
12:11396601-11436500
22:20999401-21300400
exit
EOF
```

Another example:
```
awk '{ print $2 } END { print "exit" }' calls.cnvnator | ./cnvnator -root NA12878.root -genotype 100
```

## Contact Us

Please send your comments and suggestions to abyzov.alexej@mayo.edu