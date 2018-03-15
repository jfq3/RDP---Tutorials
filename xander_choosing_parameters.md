## Choosing Xander Parameters

Parameters for running Xander are set in the configuration file (`xander_setenv.sh`) and, for submitted jobs, in the qsub command for submitting the jobs. There are comments in the configuration file and in the `README` file on [GitHub](https://github.com/rdpstaff/Xander_assembler) that explain at least in part how to select the parameters. There are three sections of the configuration file:

1. [__Directory assignment__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#directory-assignment): a user's paths for data, output results, and to programs Xander calls. 
2. __Sample naming__: a unique sample identifier to be appended to the beginning of all output files and contig id (ie fasta headers).
3. __Parameters__: defines parameters important to the quality of Xander results. 
     * [__Build__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#build-parameters): parameters for building the De Bruijn graph. These relate to the size of the data file.
     * [__Contig search__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#contig-search-parameters): parameters for searching for contigs. These impact the timing of search and are not adjusted often.
     * [__Contig merge__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#contig-merge-parameters): parameters for assembling contigs. These can impact the length and quality of assembled sequences.
     * [__Contig clustering__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#contig-clustering-parameters): Parameters for clustering resulting contigs. 
     * [__Other__](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md#other-parameters)

--- 

### Directory assignment
This section must be modified to match your file structure.

* `SEQFILE`: absolute path to sequence file. 
    * accepted file formats: fasta, fataq or gz format
    * can use wildcards (`*`) to point to multiple files as long as there are no spaces in the names
* `WORKDIR`: absolute path to working directory. It is useful to have a separate working directory for each sample of interest.
* `REF_DIR`: absolute path to `Xander_assembler` directory
* `JAR_DIR`: absolute path to `RDPTools`
* `UCHIME`: absolute path to `uchime`
* `HMMALIGN`: absolute path to `hmmalign`

---

### Build Parameters

As the name suggests, these determine how the De Bruijn graph, or bloom filter, is built. It is important to get this right because everything else depends on the quality of the boom filter. Some experimentation in selecting the parameters may be necessary, so in practice it is best to first run Xander with only the build step.

__De Bruijn graph set up__

* `FILTER_SIZE`: This value depends on the size of the data file. Based on our experience with metagenomic data from soil samples, the following choices are appropriate:

| Data file size |       |FILTER_SIZE parameter |
| :------------: |  ---- | :------------------: |
|    2 GB        |       |         32           |
|    6 GB        |       |         35           |
|   70 GB        |       |         38           |
|  350 GB        |       |         40           |

* `K_SIZE`: The kmer size (in base pairs) used for contig assembly. It must be a multiple of 3 (3 nucleotides code for an amino acid) and cannot be larger than 63; a value of 45 is recommended.

* `MIN_COUNT`: The minimum kmer occurrence in the `SEQFILE` (data file) for the kmer to be included in the final bloom filter. This is almost always equal to 1. Larger values require more memory (java heap size).

* `MAX_JVM_HEAP (java heap size)`: The maximum amount of memory allowed for the build processes. 
   * Must be larger than the size of the bloom filter, which is determined by the values of `FILTER_SIZE` and `MIN_COUNT`
   * If `MIN_COUNT` is 1, the size of the bloom filter is approximately (2<sup>(**FILTER_SIZE**-3)</sup>)/10<sup>9</sup> GB. For example:

| Filter_SIZE parameter |Approximate bloom file size |
| :-----: | :-----------: |
|    35   |      4 GB     |
|    36   |      8 GB     |
|    37   |     16 GB     |
|    38   |     32 GB     |

  * If `MIN_COUNT` is 2, then the bloom filter is approximately twice as large. 

__De Bruijn graph quality__

The quality of the bloom filter can be evaluated by examining the false discovery rate reported on the next to last line of the output file `knn_bloom_stat.txt` found in the `knn` sub-directory of the data output directory. `knn` is the kmer size specified by the `K_SIZE` parameter (i.e. k45). 
* The false discovery rate should be less than 0.01 (1 %) and depends on the parameter `FILTER\_SIZE`. 
* If the false discovery rate is greater than 0.01, delete the `knn` directory and then re-run Xander build with a larger `FILTER_SIZE`.

---

### Contig Search Parameters
These impact the timing of search and are not adjusted often.

* `PRUNE` the search if the score does not improve after the specified value for  `n_nodes`. The recommended value is 20. If this is set to 0, pruning is disabled but required memory and time increases.

* `PATHS` is the number of paths to search for each starting kmer. A value of 1 returns the shortest path.

* `LIMIT_IN_SECS` is the time limit in seconds to spend searching for each kmer. The recommended value is 100 seconds if `PATHS` = 1. If `PATHS` is larger, then the value for `LIMIT_IN_SECS` needs to be increased.

---

### Contig Merge Parameters
These can impact the length and quality of assembled sequences.

* `MIN_BITS` is the minimum assembled contigs bit score. The recommended value is 50. This value can be increased if low quality sequences are assembled. 

* `MIN_LENGTH` is the minimum length for assembled protein contigs. The recommended value is 150, which would result in a minimum assembled bp length of 450 and a minimum aa length of 150.

---

### Contig Clustering Parameters

* `DIST_CUTOFF` is the distance at which to cluster aa sequences. The recommended value is 0.01, which would cluster final contigs at 99% aa identity.

---

### Other parameters

* `THREADS` is the number of computer cores to use. 
  * Only one core is used to build the bloom filter, `THREADS` does not impact this step.
  * The find and search steps may be run in parallel, one core for each gene, as explained in the files *xander\_local\_example.md* and *xander\_hpcc\_example.md*. 
    * Set **THREADS** to the number of genes you are searching for, but do not exceed one less than the number of cores you have on your computer. 
    * For example when submitting a job to MSU's cluster, the value of ppn should be **THREADS** plus 1. 

* `NAME`=k$K_SIZE need not be changed. It defines the name of the sub-directory to which results are written (`knn`).
