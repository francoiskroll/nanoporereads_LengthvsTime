# Nanopore reads: Length vs Time

I am working on a sequencing project using the Oxford Nanopore MinION sequencing device and wanted to explore potential bias of read lengths over time in my data (for example: are short fragments sequenced first?). I thus wrote this short command line/R script to create a read length vs time since start scatterplot. For instance, it helped me decide how long I should run my flowcell to get enough long reads.

There is clearly room for improvement. But I thought it might be useful for other users of the Nanopore! Feel free to use/modify it.

I am using `albacore` (https://community.nanoporetech.com/downloads) for basecalling from the fast5 files, and I am usually putting all my reads in one single fastq file.

If all your reads are not in one single fastq file, you can concatenate all of them (assuming they are all in the same folder) with:

`cat *.fastq > pool.fastq`

# In command line
## Create list of read lengths
`cat pool.fastq | awk '{if(NR%4==2) print length($1)}' > lengths.txt`<sup> 1</sup>

## Create list of sequencing dates/times
`grep -E -o "start_time.{0,20}" pool.fastq | cut -c 12- > times.txt` <sup> 2</sup>

# In R
 R script attached.
 
 (1) From http://onetipperday.sterding.com/2012/05/simple-way-to-get-reads-length.html
 (2) From https://stackoverflow.com/questions/8101701/grep-characters-before-and-after-match
