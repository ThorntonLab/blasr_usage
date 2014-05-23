blasr_usage
===========

Aligning PACbio data to a reference.  (Specific for data received from UCI core)


The data files
======

From the core, you get:

1. file.h5.gz = the "movie".  To be used in the SMRT pipeline.  (We can typically ignore this)
2. file-Reads.fastq.gz = the reads in FASTQ format
3. file-CCS-Reads.fastq.gz = the reads sequences > 1 time.  Higher quality, fewer reads
4. file-extra.tar.gz = this is the raw data that you will use for alignment.

Align to a reference using blasr on HPC
====

Assumes that data are in the folder from which this script is submitted.  And, file-extra.tar.gz has been uncompressed, and the output is in a folder called extra.

Usage:

```
qsub blasrit.sh -q bio,pub64 `pwd` reference
```

```{sh}
#!sh

#$ -pe openmp 16-64

module load krthornt/blasr/blasr
module load samtools

cd $1/extra

sample=`basename $1`

BFILES=""
for i in *.bax.h5
do
    n=`basename $i .bax.h5`
    blasr $i $2 -bestn 100 -sam -nproc $CORES |grep -v Warning |samtools view -bS - | samtools sort - $n
    BFILES=$BFILES" "$n.bam
done

samtools merge $sample.bam $BFILES
samtools depth $sample.bam | gzip > $sample.depth.gz
```
