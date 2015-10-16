# Description

TagDigger is a program for processing FASTQ files from genotyping-by-sequencing (GBS) or restriction site-associated DNA sequencing (RAD-seq) experiments.  Its purpose is to rapidly find and count tags of known sequence that are specified by the user.  The assumption is that tags of interest have already been identified by other SNP-mining software, and now the user wants to find those same tags in other sequence data.  Although TagDigger is not graphical software, it is designed to be accessible to people without programming experience.  TagDigger also uses very little RAM and can process a 200 million read FASTQ file in a couple hours on a laptop computer.

If the software does not seem to be functioning properly, please file an Issue or otherwise notify me.

# Obtaining and running the software

## Requirements

[Python 3](http://www.python.org) needs to be installed.  TagDigger will not work with Python 2.  Python and TagDigger will work on any operating system.

Although not required, on Windows it is helpful to uncheck "Hide extensions for known file types" in "Folder options".

## Download

Click "Download ZIP" on the right hand side of the GitHub page for TagDigger.  Extract the zip archive on your computer.  No installation is necessary.

Of course, if you are familiar with Git you can use Git to obtain the software.

## Running the software

On Windows, TagDigger can be launched simply by double-clicking the file "tagdigger_interactive.py".

Alternatively, on any operating system, you can use the shell or command prompt to launch TagDigger.  Make sure your PATH variable includes the directory where the Python 3 executable is located.  Then in the shell, `cd` (change directory) to the directory where the TagDigger Python files (the files in this GitHub repository) are located, and type `python tagdigger_interactive.py`.

The file "tagdigger_fun.py" must be in the same folder as "tagdigger_interactive.py" for the program to work.

If you find that you have made a mistake in your input and can't go back, simply close the window running TagDigger and re-launch the program.

# Input files

## FASTQ

TagDigger reads FASTQ files generated by any modern high-throughput DNA sequencing technology.  If the file is compressed with gzip, it should end with the file extension ".gz".  If the file is not compressed, it should NOT end in ".gz".  Beyond that there are no rules for naming the FASTQ files.

## Barcode key file

The key file indicates the names of FASTQ files that should be read, the barcodes that should be searched for in each file, and the sample name that is associated with each barcode.  It should be in CSV format (comma delimited), as can be produced by LibreOffice Calc or MS Excel.  There should be a header containing the column names "File", "Barcode", and "Sample".  Other columns can be included and will be ignored by the software.  A short example is below.

```
File,Barcode,Sample,
lib01.fasta.gz,AACG,PI230189,
lib01.fasta.gz,TTGACC,KD-230-a,
my-second-lib_fasta.txt,CCGA,foo-2705,
```

If your FASTQ files have already been split by barcode AND the barcode sequence is removed, you can leave the Barcode column blank and simply list files and sample names.

## Tag file

Tag sequences can be read in one of several different formats.  Regardless of format, the following must be true:

* The restriction site does not need to be present in the tag sequences provided by the user.  However, if it is present in some tags, it must be present in all.
* Marker names must not contain underscores.

Formats that code for biallelic markers will label the sequence tags in each pair as "0" or "1".  These labels can then be used when the SNPs are converted to numeric format.  A homozygote for the 0 allele is coded as 0, a homozygote for the 1 allele is coded as 2, and a heterozygote is coded as 1.

For all formats, the user can optionally supply a list of markers to retain from the tag file.  (For example, if you have thousands of markers in your tag file, but you just want to examine a few dozen.)  The list can be supplied in a plain text or CSV file with one marker name per line, and no other content.

Example list of markers:

```
TP276
TP1003
TP1206
```

### UNEAK FASTA

[TASSEL](http://www.maizegenetics.net/tassel)'s UNEAK pipeline produces a FASTA file in the following format that can be read by TagDigger:

```
>TP276_query_64
TGCAGAAAAAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT
>TP276_hit_64
TGCAGAAACAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT
>TP539_query_64
TGCAGAAAAAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT
>TP539_hit_64
TGCAGAAATAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT
```

The number at the end of each comment line indicates the length of the tag, and is used by TagDigger to trim off poly-A padding that is added by UNEAK to tags containing a restriction cut site.

TagDigger will also find the SNP for each tag pair and include the nucleotide in the tag name.  Tags are assigned to be '0' or '1' based on alphabetical order of the SNP alleles, to be consistent with the [hapMap2numeric and hapMap2genlight](http:www.github.com/lvclark/R_genetics_conv) functions.

### Merged tags

Tags can also be read in the following CSV format, which saves computer memory and makes it easy for a human eye to see the SNP(s):

```
Marker name,Tag sequence,
TP276,TGCAGAAA[A/C]AAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,
TP539,TGCAGAAA[A/T]AAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,
```

The nucleotides before and after the forward slash will be the 0 and 1 alleles, respectively.

Polymorphic regions can be more than one nucleotide long, but there can only be one pair of square brackets and one forward slash in the sequence.  For example:

```
Mrkr2010,ACGTAAACGATA[AAG/GAC]TACGATAAATTT,
```

### Tags in columns

Two alternative tag sequences for one marker can be arranged in columns of a CSV:

```
Marker name,Tag sequence 0,Tag sequence 1,
TP276,TGCAGAAAAAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,TGCAGAAACAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,
TP539,TGCAGAAAAAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,TGCAGAAATAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,
```

### Tags in rows

Lastly, each tag can be in its own row in a CSV file.  This is the only format that allows a number of tags per marker other than two.  Alleles can have any name, but the names '0' and '1' will facilitate the use of other TagDigger tools for biallelic markers.

```
Marker name,Allele name,Tag sequence,
TP276,0,TGCAGAAAAAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,
TP276,1,TGCAGAAACAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,
TP539,0,TGCAGAAAAAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,
TP539,1,TGCAGAAATAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,
Mrker2035,dom,AGCTAGACTAGGGTTACCAGTACTTACCGATACATTAAAGCATCAT,
Mrker4050,0,AGTAGGGAAAGGCCGGTAAGGCAACTAAA,
Mrker4050,1,AGTAGGGAGAGGCCGGTAAGGCAACTAAA,
Mrker4050,2,AGTAGGGAAAGGCCGGCAAGGCAACTAAA,

```

# Output

If multiple barcodes have the sample name within and/or among libraries, the read counts will be added together for all identically-named samples.

A CSV file of read counts is output, with samples in rows and tags in columns.

If tags represent pairs of binary SNPs, with alleles labeled '0' or '1', a CSV of numeric diploid genotypes (0, 1, 2) can optionally be written, with samples in rows and markers in columns.

# Barcode splitter

TagDigger also includes a program for splitting one FASTQ file into multiple files according to barcode.  FASTQ files can be in uncompressed or gzip format as described above.  The barcode key file should also be in the format described above, but with column headers "Input File", "Barcode", and "Output File".  Download this repository and double-click "barcode_splitter.py" to launch the program.

Although the main TagDigger program works with libraries from any enzyme combination, the barcode splitter currently only supports PstI-MspI and NsiI-MspI.  Available adapter sequences include those from [Poland *et al.* (2012)](http://dx.doi.org/10.1371/journal.pone.0032253) and those used in the [Sacks lab](http://openwetware.org/wiki/Sacks:RAD-seq), designed by Megan Hall.

The output FASTQ files are uncompressed, with barcodes, adapter sequence, and potentially chimeric sequence clipped out.  The comment line for each read has the barcode appended to it.

Optionally, the barcode splitter can also generate a CSV file listing MD5 checksums for each output FASTQ file.

# Tag Manager

A third program included with TagDigger, "tag_manager.py", can be used for creating universal names for markers across multiple projects.  It can read in tag sequences in any of the four formats listed above.  Tag sequences are output in the "merged" format (e.g. `AACG[C/T]CCA`) in a CSV, with new marker names consisting of a user-specified prefix followed by a number.  The original marker names can optionally be included in the output, along with any other columns of data that the user provides in a separate CSV file.  For example, input from the the following files:

```
>TP276_query_64
TGCAGAAAAAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT
>TP276_hit_64
TGCAGAAACAAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT
>TP539_query_64
TGCAGAAAAAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT
>TP539_hit_64
TGCAGAAATAAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT
```

and

```
Marker name,Allele frequency,
TP276,0.12,
TP539,0.05,
```

could be used to produce the output:

```
Marker name,Tag sequence,Allele frequency,Name in study 1,
MyLab00001,TGCAGAAA[A/C]AAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,0.12,TP276,
MyLab00002,TGCAGAAA[A/T]AAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,0.05,TP539,
```

Tag Manager can also export all markers to a FASTA file, using IUPAC codes for ambiguous nucleotides.  This FASTA file can be aligned to a reference genome using software such as Bowtie2, and the resulting SAM file can be imported by Tag Manager to add new columns to the database for chromosome, position, and alignment quality score.

Of course, after generating the initial marker database, additional tag files can be imported.  Markers that match the sequence of existing markers will be identified as those markers (in an optional column to list the original marker names), and markers with new sequence will be added to the bottom of the list.  For example:

```
Marker name,Tag sequence,Allele frequency,Name in study 1,Name in study 2,
MyLab00001,TGCAGAAA[A/C]AAAAATCACAGCACAGGCACTAGAAGCACTGGTAGTAACTCGAGACAGGATGTAT,0.12,TP276,,
MyLab00002,TGCAGAAA[A/T]AAACTTGAGAAAGGCCGTACTTTTAAAGTGTATTATAGAAAAATCTTAGGTGCAT,0.05,TP539,TP1001,
MyLab00003,TGCAGAGAATATAATCATCACCTGGGCGCTCGCTCAACTC[A/G]ACGAAAGGCGCAGACTTTCCGAC,,,TP2002,
```
