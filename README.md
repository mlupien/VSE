# VSE
VSE is a Perl/Rscript command line tool to calculate the enrichment of associated variant set (AVS) for an array of genomic regions.

**Program requirement:**
```
- Perl (5.18 or higher)
- R (3.1.1 or higher)
- bedtools (2.2.4 or higher) and must be globally executable
```
**Perl module required:**
```
- File::Basename
```
**R package required:**
```
- ggplot2
- reshape
- car
```
####Installation:
**Step 1: Install Perl**

  You can get the Perl software from [their website](https://www.perl.org/get.html) and install. Version 5.x is required.
  You must also install the Perl module ```File::Basename``` (module's CPAN page is [here]( http://search.cpan.org/~nwclark/perl-5.8.6/lib/File/Basename.pm)).

  In most cases, the following steps will work.
  
  1. First, install cpanMinus if not installed:

    ```sudo cpan App::cpanminus```

  2. Then install File::Basename by the following command:

    ```cpanm File::Basename```

  If you have issues, you can refer to [CPAN help page](http://www.cpan.org/modules/INSTALL.html) for more details.

**Step 2: Install R**

R is also required for calculating the statistics and generating the plots. You can download R at [www.r-project.org](http://www.r-project.org). Version 3.1.1 or greater is required.

The following R packages are required: ```ggplot2```,```reshape```,```car``` , all of which can be downloaded and installed from CRAN.

In most cases, you can install the packages by:

1. Go to R command line by typing ```R``` in your terminal
2. type

  ```
  install.packages("ggplot2")  
  install.packages("reshape")  
  install.packages("car")
  ```

3. Exit R environment by typing ```q()```

```Rscript``` should be an executable command from any location.

**Step 3: Install bedtools**

You can download and install the latest version of bedtools from [their github repository](https://github.com/arq5x/bedtools2). 

In most cases, you can install by this:

1. In terminal, go to downloaded bedtools folder:

  ```
  cd bedtools-2.25.0
  ```

2. Compile the tool by typing

  ```
  make
  ```

3. Copy the binary programs from ```bin/``` to your executable directory, which is usually ```/usr/bin/``` or ```/usr/local/bin```

  ```
  sudo cp bin/* /usr/bin/
  ```

If this does not work, further installation instruction can be found  [in their website](http://bedtools.readthedocs.org/en/latest/content/installation.html)

The ```intersectBed``` should be an executable command from any location.

**Step 4: Download VSE**

You can just download and run ```VSE.pl```, no other installation required if you have the required programs already installed. The directory structure must be intact; i.e., ```lib``` and ```data``` directories must reside in the same directory as ```VSE.pl```.


###Using VSE
#####Example command:
```
perl VSE.pl -f example.SNPs/NHGRI-BCa.bed \
            -l example.SNPs/ld_BCa.bed \
            -s run1 -d example.beds \
            -v
```

#####Options:
```
-f bed file for tagSNPs
-l file containing LD SNPs information
-d full path to directory containing all bed files or path to a single bed file
-s output directory suffix. Output files will be saved in suffix.output directory
-r [0.6/0.7/0.8/0.9/1] r2 value; default 0.8
-p [all/AVS/MRV/xml/R] modular run; default all
-A Suffix for existing AVS/MRV files. Only functional when -p is xml.
-h help
-v verbose
```
####Input
VSE requires three input files.
######The tagSNPs:
It should be a standard tab delimited bed file and provided with ```-f``` parameter. There should be no header line.
Example:
```
chr1  1000  1001  rs00001
```
######The LD SNPs
The list of LD SNPs should be a **tab delimited** bed file in the following format and should be provided with ```-l``` parameter. 
```
chr1  900 901 LDSNPid tagSNPid other_optional_columns
```
There should be no header line. The LD SNPs must be calculated based on EUR population of 1000 Genome Project Phase III genotyping data (Release 2013/05). It's important to use the updated genotyping information for calculating LD because the newer releases are more accurant than the older ones because of inclusion of more individuals. However, for older studies, VSE will soon support 1000 Genome Project Phase I data as well.

######The genomic ranges
The genomic regions should be a single directory and the directory path should be given in ```-d``` parameter. The genomics regions should be bed files. The name of the file is used as labels for the regions. For examples, ```DHS.bed``` will be denoted as DHS in final figures and table.
```-d``` can also be a path to a bed file. In that case, the tallies will be printed on screen and the enrichment analysis will not be run.

####Output
VSE produces multiple output files in suffix.output directory.

```suffix.density.pdf``` shows the density of overlapping tallies from the null

```suffix.VSE.stat.txt``` contains the statistics table

```suffix.final_boxplot.pdf``` visualizes the null distribution and enrichment of AVS

```suffix.matrix.pdf``` binary representation of overlapping between each locus and annotation. Overlapping is defined as at least one SNP (associated or linked) is within the annotation.

```suffix.VSE.txt``` is a matrix of all overlapping tallies by AVS and MRVS. The first column is the AVS tally and the rest are MRVS.

####Running VSE
VSE can be run in different parts using ```-p``` parameter. For the first run, ```-p all``` or no ```-p``` is recommended. However, once you create the AVS and MRVS for a set of SNPs, you can use ```-p xml``` and ```-p R``` for checking the enrichment of the SNPs over new genomic ranges. Below are examples of certain situations and command lines that to be used:
######Running for the first time for a set of variants:
```perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/histone_marks/ -s run1 -v```
######Running the same set of SNPs but for a different batch of genomic regions:
```perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p xml -A run1 -s run2 -v```

This command line will use the AVS and MRVS outputted from run1 and will produce new matrix file in ```run2.output``` directory. Then you can run ```-p R``` to compute enrichment and generate the plots:

```perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p R -A run1 -s run2 -v```
######Running for just one genomic region file:
```perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/POL2_binding.bed -s run-pol2 -v```

This will output the overlapping tallies for AVS and MRVS for POL2 on the screen. You can copy this line to any other *.VSE.txt file from other experiments. For example, you can add the line to ```run2.output/run2.VSE.txt``` from the TF_binding analysis (run2). You can then run ```-p R``` to redo the enrichment analysis, now including POL2: ```perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p R -A run2 -s run2_with_pol2 -v```

#####There are several factors to consider:
1. VSE is sensitive to the number of tagSNPs. From our trial and error tests, too low number of tagSNPs (below 15) provide imprecise result.
2. The quality of ChIP-seq data is very important. We recommend users to confirm the quality of the ChIP-seq data and to only use data that are of good quality to avoid false enrichment. There are tools like ChIPQC or Chillin for quality control of ChIP-seq data.
3. Make sure that you use the same r2 cutoff that you used to determine your LD SNPs. Also, the LD SNPs must be calculated using 1000 Genome Project Phase III (May 2013) release.
