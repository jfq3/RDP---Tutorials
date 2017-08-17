## Choosing Xander Parameters

Parameters for running Xander are set in the configuration file and, for submitted jobs, in the qsub command for submitting the jobs. There are comments in the configuration file and in the README file on [GitHub](https://github.com/rdpstaff/Xander_assembler "Xander") that explain at least in part how to select the parameters. Those in the first section of the configuration file are merely the paths to use to find data, to output results, and to programs Xander calls. The second section sets a sample identifier to be appended to the beginning of the contig names and gives the names of the genes being assembled. Parameters important to the quality of Xander results begin in the third section. Of these, usually the only ones that need to be changed from their recommended values are **FILTER_SIZE**, **MAX_JAR_HEAP**, and **THREADS**.

### Build Parameters

The third section of the configuration script begins with build parameters. As the name suggests, these determine how the De Bruijn graph, or bloom filter, is built. It is important to get this right because everything else depends on the quality of the boom filter. Some experimentation in selecting the parameters may be necessary, so in practice it is best to first run Xander with only the build step.

The quality of the bloom filter can be evaluated by examining the false discovery rate reported on the next to last line of the output file *knn\_bloom\_stat.txt* found in the *knn* sub-directory of the data output directory.  Here the *nn* in *knn* is the kmer size specified by the **K_SIZE** parameter. The false discovery rate should be less than 0.01 (1 %) and depends on the parameter **FILTER\_SIZE**. If the false discovery rate is greater than 0.01 then the **FILTER_SIZE** should be increased.

The value for **FILTER_SIZE** depends on the size of the data file. Based on our experience with metagenomic data from soil samples, the following choices are appropriate:

| Data file size |       |FILTER_SIZE parameter |
| :------------: |  ---- | :------------------: |
|    2 GB        |       |         32           |
|    6 GB        |       |         35           |
|   70 GB        |       |         38           |
|  350 GB        |       |         40           |

**K_SIZE** is the kmer size used for contig assembly. It must be a multiple of 3 (3 nucleotides code for an amino acid) an cannot be larger than 63; a value of 45 is recommended.

**MIN_COUNT** is the minimum kmer occurrence in the **SEQFILE** (data file) for the kmer to be included in the final bloom filter. This is almost always equal to 1. Larger values require more memory (java heap size).

**MAX_JVM_HEAP**  or java heap size is the maximum amount of memory allowed for the build processes. It must be larger than the size of the bloom filter, which is determined by the values of **FILTER_SIZE** and **MIN_COUNT**. If **MIN_COUNT** is 1, the size of the bloom filter is approximately (2<sup>(**FILTER_SIZE**-3)</sup>)/10<sup>9</sup> GB. For example:

| Filter_SIZE parameter |Approximate bloom file size |
| :-----: | :-----------: |
|    35   |      4 GB     |
|    36   |      8 GB     |
|    37   |     16 GB     |
|    38   |     32 GB     |

If **MIN_COUNT** is 2, then the bloom filter is approximately twice as large. 

### Contig Search Parameters

**PRUNE** the search if the score does not improve after the specified value for  **n_nodes**. The recommended value is 20. Set this to 0 to disable pruning.

**PATHS** is the number of paths to search for each starting kmer. A value of 1 returns the shortest path.

**LIMIT_IN_SECS** is the time limit in seconds to spend searching for each kmer. The recommended value is 100 seconds if **PATHS** = 1. If **PATHS** is larger, then the value for **LIMIT_IN_SECS** needs to be increased.

### Contig Merge Parameters

**MIN_BITS** is the minimum assembled contigs bit score. The recommended value is 50.

**MIN_LENGTH** is the minimum length for assembled protein contigs. The recommended value is 150.

### Contig Clustering Parameters

**DIST_CUTOFF** is the distance at which to cluster aa sequences. The recommended value is 0.01.

### Other parameters

**THREADS** is the number of computer cores to use. Only one core is used to build the bloom filter. The find and search steps may be run in parallel, one core for each gene, as explained in the files *xander\_local\_example.md* and *xander\_hpcc\_example.md*. In this case, set **THREADS** to the number of genes you are searching for, but do not exceed one less than the number of cores you have on your computer. When submitting a job to MSU's cluster, the value of ppn should be **THREADS** plus 1. 

**NAME**=k$K_SIZE need not be changed. It defines the name of the sub-directory to which results are written.