# Nanopore reads: Length vs Time

I am working on a sequencing project using the Oxford Nanopore MinION sequencing device and wanted to explore potential bias of read lengths over time in my data (for example: are short fragments sequenced first?). I thus wrote this short command line/R script to create a read length vs time since start scatterplot. For instance, it helped me to decide how long I should run my sequencing runs to get longer reads.

There is clearly room for improvement. But I thought it might be useful for other users of the Nanopore! Feel free to use/modify it.

I am using `albacore` (https://community.nanoporetech.com/downloads) for basecalling from the fast5 files, and I am usually putting all my reads in one single fastq file.

If all your reads are not in one single fastq file, you can concatenate all of them (assuming they are all in the same folder) with:

`cat *.fastq > pool.fastq`

# In command line
## Create list of read lengths
`cat pool.fastq | awk '{if(NR%4==2) print length($1)}' > lengths.txt` (1)

## Create list of sequencing dates/times
`grep -E -o "start_time.{0,20}" pool.fastq | cut -c 12- > times.txt` (2)

# In R
```
library (ggplot2)
library (hexbin)

setwd("~/Documents/nanopore_runs/3_7_TD65_primers56/workspace")
lgths <- read.csv ('lengths.txt')
dates_times <- read.csv ('times.txt')

dates <- apply (dates_times, 1, function (x) gsub ('T.*', '', x))
times <- apply (dates_times, 1, function (x) gsub ('.*T', '', x))

# convert times to a numeric
hours <- (as.numeric (substr (times, 1, 2))) * 3600
minutes <- (as.numeric (substr (times, 4, 5))) * 60
seconds <- (as.numeric (substr (times, 7, 8)))

rawtime <- cbind (hours, minutes, seconds)
rawtimes2 <- apply (rawtime, 1, sum)

# when did it start
start_time <- min (rawtimes2) # seconds after midnight
times2 <- rawtimes2 - start_time

timelgth <- cbind (lgths, times2)
colnames (timelgth) <- c('length', 'time')
lgth_kb <- timelgth$length / 1000
time_hr <- timelgth$time / 3600
timelgth <- cbind (timelgth, lgth_kb, time_hr)
colnames (timelgth) <- c('length', 'time', 'length_kb', 'time_hr')

# # tmp so it renders more quickly
# timelgth <- timelgth [sample(nrow(timelgth), 3000), ]

# linear regression
lm_timelgth <- lm (length_kb ~ time_hr, timelgth)
intercept <- lm_timelgth$coefficients [1]
coefficient <- lm_timelgth$coefficients [2]

plot (x = timelgth$time, y = timelgth$length, 
      pch = 20, bty = 'l',
      xlab = 'time (seconds)', ylab = 'length (bp)')

abline (intercept, coefficient)

ggplot (timelgth, aes(x = time_hr, y = length_kb)) + 
  geom_point (alpha = 0.2, shape = 20) +
  geom_abline (intercept = intercept, slope = coefficient, color = '#9E0000', size = 1.5) +
  xlab ('Time (hours)') +
  ylab ('Length (kb)')

# hexagons heatmap
# ggplot (timelgth, aes(x = time_hr, y = length_kb)) + 
#   stat_binhex () +
#   xlab ('Time (hours)') +
#   ylab ('Length (kb)')
# 
# rectangles heatmap
# ggplot (timelgth, aes(x = time_hr, y = length_kb)) +
#   geom_bin2d () +
#   xlab ('Time (hours)') +
#   ylab ('Length (kb)')
```
