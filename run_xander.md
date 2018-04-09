## Running Xander

Two script files are required to run Xander. These are the configuration script and the command script. The configuration file provides paths and parameter values to be used. The command script parses the command you enter for the steps to be run and the genes to be found, and then executes the Xander steps specified. It also includes commands for several post processing steps that summarize the results.

Templates for these two script files are part of the RDPTools installation and are located in the `RDPTools/Xander_assembler/bin` sub-directory. The configuration script file is named `xander_setenv.sh` and the command script file is named `run_xander_skel.sh`. The paths in these files are appropriate to the installation of RDPTools on MSU's HPCC. The paths must be edited for use with another installation of RDPTools.

### The Xander Exercises

There are three exercises in this section. The first two are run interactively, either locally or on MSU's HPCC, and the third demonstrates how to submit Xander jobs to the HPCC. Gain experience editing the command and configuration files by doing one (or both) of the interactive exercises. Analysis of the toy data set included with Xander takes only a few minutes to run, so it is permissible to run it interactively on the HPCC. Analysis of real data takes long enough that such jobs must be submitted to the queueing system.

### The Xander Script

For your reference, the command and configuration scripts are given below. These are identical to the ones in the Xander installation except that comment lines (e.g. ## First section, etc.) have been inserted in the configuration script to identify the sections discussed in this manual.

#### The Configuration Script (`xander_setenv.sh`)


    #!/bin/bash -login
    
    #### start of configuration
    
    ###### Adjust values for these parameters ####
    #       SEQFILE, SAMPLE_SHORTNAME
    #       WORKDIR, REF_DIR, JAR_DIR, UCHIME, HMMALIGN
    #       FILTER_SIZE, MAX_JVM_HEAP, K_SIZE
    #       THREADS
    #####################
    
    ## First section
    ## THIS SECTION MUST BE MODIFIED FOR YOUR FILE SYSTEM. MUST BE ABSOLUTE PATH
    ## SEQFILE can use wildcards to point to multiple files (fasta, fastq, or 
	## gz format), as long as there are no spaces in the names.
    SEQFILE=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata/test_reads.fa
    WORKDIR=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata
    REF_DIR=/mnt/research/rdp/public/RDPTools/Xander_assembler
    JAR_DIR=/mnt/research/rdp/public/RDPTools
    UCHIME=/mnt/research/rdp/public/thirdParty/uchime-4.2.40/uchime
    HMMALIGN=/opt/software/HMMER/3.1b1--GCC-4.4.5/bin/hmmalign
    
    ## Second section
    ## THIS SECTION NEEDS TO BE MODIFIED, SAMPLE_SHORTNAME WILL BE THE PREFIX
	## OF THE CONTIG ID
    SAMPLE_SHORTNAME=test
    
    ## Third section
    ## THIS SECTION MUST BE MODIFIED BASED ON THE INPUT DATASETS
    ## De Bruijn Graph Build Parameters
    K_SIZE=45  # kmer size, should be multiple of 3
    FILTER_SIZE=32 # memory = 2**FILTER_SIZE, 38 = 32 GB, 37 = 16 GB, 36 = 8 GB, 35 = 4 GB, increase FILTER_SIZE if the bloom filter predicted false positive rate is greater than 1%
    MAX_JVM_HEAP=2G # memory for java program, must be larger than the corresponding memory of the FILTER_SIZE
    MIN_COUNT=2  # minimum kmer abundance in SEQFILE to be included in the final de Bruijn graph structure
    
    ## number of threads to use for find starting kmer step and kmer coverage mapping step
    THREADS=1
    
    ## Contig Search Parameters
    PRUNE=20 # prune the search if the score does not improve after n_nodes (default 20, set to -1 to disable pruning)
    PATHS=1 # number of paths to search for each starting kmer, default 1 returns the shortest path
    LIMIT_IN_SECS=100 # number of seconds a search allowed for each kmer, recommend 100 secs if PATHS is 1, need to increase if PATHS is larger
    
    ## Contig Merge Parameters
    MIN_BITS=50  # mimimum assembled contigs bit score
    MIN_LENGTH=150  # minimum assembled protein contigs
    
    ## Contig Clustering Parameters
    DIST_CUTOFF=0.01  # cluster at aa distance
    
    NAME=k${K_SIZE}
    
    #### end of configuration

#### The Command Script (`run_xander_skel.sh`)

    
    #!/bin/bash -login
    
    ## This is the main script to run Xander assembly
    
    BASEDIR=/mnt/research/rdp/public/RDPTools/Xander_assembler/bin # for MSU's HPCC
    
    if [ $# -ne 3 ]; then
        echo "Requires three inputs : /path/xander_setenv.sh tasks genes"
	    echo "  xander_setenv.sh is a file containing the parameter settings, requires absolute path. See example RDPTools/Xander_assembler/bin/xander_setenv.sh"
	    echo '  tasks should contain one or more of the following processing steps with quotes around: build find search"'
	    echo '  genes should contain one or more genes to process with quotes around'
	    echo 'Example command: /path/xander_setenv.sh "build find search" "nifH nirK rplB"'
        exit 1
    fi
    
    #### start of configuration
    ENVFILE=$1
    tasks=$2
    genes=$3
    source $ENVFILE
    #### end of configuration
    
    ## build bloom filter, this step takes time, not multithreaded yet, wait for future improvement
    ## only once for each dataset at given kmer length
    if [[ " ${tasks[*]} " == *"build"* ]]; then
    	$BASEDIR/run_xander_build.sh $ENVFILE || { exit 1; }
    fi
    
    ## find starting kmers, multiple genes should be run together to save time, has multithread option
    if [[ " ${tasks[*]} " == *"find"* ]]; then
	    $BASEDIR/run_xander_findStarts.sh $ENVFILE "$genes" || { exit 1; }
    fi
    
    ## search contigs and post-assembly processing
    ## can run in parallel 
    
    if [[ " ${tasks[*]} " == *"search"* ]]; then
        for gene in ${genes[*]}
        do
        	$BASEDIR/run_xander_search.sh $ENVFILE "${gene}"  
        done
    fi
