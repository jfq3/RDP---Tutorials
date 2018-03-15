## Combining Xander Results from Multiple Samples

To perform any type of community analysis with  your assembled contigs, the Xander steps Build, Find, and Search must be performed for each sample in your experiment. After Build, Find, and Search are complete on multiple samples, they can be compared. 

The script `get_OTUabundance.sh` in `RDPTools/Xander_assembler/bin/` is provided to create coverage-adjusted OTU abundance data matrices from contigs of the same gene from multiple samples. 
* __Inputs__
	* `final_prot_aligned.fasta` files for _all_ samples of interest
	* A file of all the sample coverage files concatenated together
	
* __Outputs__
	* `rformat_dist_0.##.txt`: data matrix files with OTU abundances for each sample at given distances (0 to 0.5 by 0.01 steps by default). The data matrices can then imported to R for more extensive analysis and visualization functions. Currently these are rounded down to include whole number OTU abundances only.

---

### Planning ahead

It is easier to collect the necessary files together if you plan ahead. 
* Before you run Xander, create a directory for your experiment. Within it, create a Xander output directory for each sample in your experiment. 
* When you run Xander, in the `xander_setenv.sh` file for each sample give a `SAMPLE_SHORTNAME` that identifies the sample and point the output to the appropriate sample directory. If you are using MSU's HPCC, you can submit multiple jobs (up to 250, but good luck with that) at the same time. 

---

### Example

If you follow the above strategy, then you can collect all `coverage.txt` and `final_prot_aligned.fasta` files together using the find command with `-exec`. For example, if the directory for your experiment is `my_experiment` in your home directory, the following will collect all `coverage.txt` and `final_prot_aligned` fasta files for `nifH` into directory `multi_nifH` in your home directory: 

    mkdir ~/multi_nifH
    find ~/my_experiment/ -name "*nifH_45_coverage.txt" -exec cp {} ~/multi_nifH/ \;
    find ~/my_experiment/ -name "*nifH_45_final_prot_aligned.fasta" -exec cp {} ~/multi_nifH/ \;

Once all the coverage and fasta files for the same gene have been collected in one directory, from that directory run the script below. You will likely need  to edit the path to RDPTools and perhaps the java heap request (-Xmx2g requests 2 gigabytes). This script is modified from `get_OTUabundance.sh` which was written  specifically to run as a submitted job on MSU's HPCC.
    
    #!/bin/bash
	# xander_cluster_samples.sh
	RDPToolsDir=/mnt/research/rdp/public/RDPTools
	# Catenate coverage files, assumed to be in the present directory.
	cat *coverage.txt > sam_coverage.txt
	#Dereplilcate
	java -Xmx2g -jar $RDPToolsDir/Clustering.jar derep -o derep.fa -m '#=GC_RF' ids samples *.fasta
	#Calculate distance matrix
	java -Xmx2g -jar $RDPToolsDir/Clustering.jar dmatrix -c 0.5 -I derep.fa -i ids -l 50 -o dmatrix.bin
	#Cluster. By default, the resulting clust file includes results for all 
	# distances from 0 to 0.5 by 0.01.
	java -Xmx2g -jar $RDPToolsDir/Clustering.jar cluster -d dmatrix.bin -i ids -s samples -o complete.clust
	# Remove unnecessary files.
    rm dmatrix.bin nonoverlapping.bin
	# Get coverage adjusted OTU matrix files
	java -Xmx2g -jar $RDPToolsDir/Clustering.jar cluster_to_Rformat complete.clust . 0 0.5 sam_coverage.txt

The last step produces OTU tables as text files, one for each distance in the cluster file. The files are tab-delimited with samples in rows and OTUs (contigs) in columns and have names of the form `rformat_dist_0.xx.txt`. They are easily imported into R for further analysis. 


 
