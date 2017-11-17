## Alternative Procedure for Submitting Xander Jobs
The file xander_hpcc_example.md, part of these tutorials, includes a section on submitting Xander jobs to MSU's HPCC. That section describes a way of submitting a job using a long one line qsub command with -l and -v flags. Some users are uncomfortable with that method, and admittedly it is prone to typographical errors. So that jobs may instead be submitted using a qsub file, I have made small changes to the xander_setenv.sh and xander_run.sh files; these changes are commented in the code below. Additionally I provide an example qsub file. All three files require some editing for paths, sequence file names and run parameters. After editing, put all three files in your working directory and make sure they are readable and executable. Then to submit the job, from your working directory all you have to do is issue the command:  

    qsub submit_xander.qsub

These instructions are specific to MSU's HPCC, but hopefully provide guidance for submitting Xander jobs on other computer clusters using the same qsub method.  

### The qsub_xander_run.sh file
Copy the code below to your editor. Give it the name qsub_xander_run.sh. Edit the assignment for the ENVFILE variable to include your working directory. The assignment for BASEDIR is correct for the RDPTools installation on MSU's HPCC. If you are using a different machine, edit this line as appropriate. Make sure the file has UNIX line endings, save your changes, and move it to your working directory on the HPCC. Make sure it is readable and executable.  

    #!/bin/bash -login

    BASEDIR=/mnt/research/rdp/public/RDPTools/Xander_assembler/bin/
    
    # I added the following line as ENVFILE was previously defined only in the qsub command line.
    ENVFILE=/mnt/home/quensenj/xander_qsub/qsub_xander_setenv.sh

    #### start of configuration
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

### The qsub_xander_setenv.sh file
Copy the code below to your editor and give it the name qsub_xander_setenv.sh. To run this as an example on MSU's HPCC, you need edit only the assignment for WORKDIR. For installations on other machines, edit the paths in the first section as appropriate. For other data, edit the the second and third sections. Make sure the file has UNIX line endings, save your changes, and move he file to your working directory. Make sure the file is readable and executable.  

    #!/bin/bash -login

    # start of configuration

    #   Adjust values for these parameters ##
    #   SEQFILE, genes, SAMPLE_SHORTNAME
    #   WORKDIR, REF_DIR, JAR_DIR, UCHIME, HMMALIGN
    #   FILTER_SIZE, MAX_JVM_HEAP, K_SIZE
    #   THREADS, ppn
    #

    # THIS SECTION MUST BE MODIFIED FOR YOUR FILE SYSTEM. MUST BE ABSOLUTE PATH
    # SEQFILE can use wildcards to point to multiple files (fasta, fastq or gz format), as long as there are no spaces in the names
    SEQFILE=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata/test_reads.fa
    WORKDIR=/mnt/home/quensenj/xander_qsub
    REF_DIR=/mnt/research/rdp/public/RDPTools/Xander_assembler
    JAR_DIR=/mnt/research/rdp/public/RDPTools
    UCHIME=/mnt/research/rdp/public/thirdParty/uchime-4.2.40/uchime
    HMMALIGN=/opt/software/HMMER/3.1b1--GCC-4.4.5/bin/hmmalign

    # THIS SECTION NEED TO BE MODIFIED FOR GENES OF INTEREST, and SAMPLE_SHORTNAME WILL BE THE PREFIX OF CONTIG ID

    # Previously the lists of tasks and genes were not inside parentheses. This called failure if more than one task or gene.
    tasks="build find search"
    genes="nifH rplB"
    SAMPLE_SHORTNAME=test

    # THIS SECTION MUST BE MODIFIED BASED ON THE INPUT DATASETS
    # De Bruijn Graph Build Parameters
    K_SIZE=45  # kmer size, should be multiple of 3
    FILTER_SIZE=32 # memory = 2**FILTER_SIZE, 38 = 32 GB, 37 = 16 GB, 36 = 8 GB, 35 = 4 GB, increase FILTER_SIZE if the bloom filter predicted false positive rate is greater than 1%
    MAX_JVM_HEAP=2G # memory for java program, must be larger than the corresponding memory of the FILTER_SIZE
    MIN_COUNT=2  # minimum kmer abundance in SEQFILE to be included in the final de Bruijn graph structure

    # ppn should be THREADS +1
    THREADS=1

    # Contig Search Parameters
    PRUNE=20 # prune the search if the score does not improve after n_nodes (default 20, set to -1 to disable pruning)
    PATHS=1 # number of paths to search for each starting kmer, default 1 returns the shortest path
    LIMIT_IN_SECS=100 # number of seconds a search allowed for each kmer, recommend 100 secs if PATHS is 1, need to increase if PATHS is large 

    # Contig Merge Parameters
    MIN_BITS=50  # mimimum assembled contigs bit score
    MIN_LENGTH=150  # minimum assembled protein contigs

    # Contig Clustering Parameters
    DIST_CUTOFF=0.01  # cluster at aa distance 

    NAME=k${K_SIZE}

    ## end of configuration

### The submit_xander.qsub file
Copy the code below to your editor and give it the name submit_xander.qsub. To run this as an example you need edit only your email address and home directory. To process your own data, you will also need to edit the resources requested and run name. Make sure the file has UNIX line endings, save your changes and move the file to your working directory. Make sure it is readable and executable.

    #!/bin/bash --login
    #PBS -l walltime=00:30:00,nodes=01:ppn=2,mem=2GB
    #PBS -j oe
    #PBS -N test_xander
    #PBS -m abe
    #PBS -M <your email address>

    # This is the file used to submit a job with the modified run and setenv files.
    # Set the number of OpenMP threads
    export OMP_NUM_THREADS=1
    cd /<your home directory>/xander_qsub

    # Begin program here.
    ./qsub_xander_run.sh


To submit the job, from your working directory enter the command:  

    qsub submit_xander.qsub
