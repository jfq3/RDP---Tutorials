## Xander Example on MSU's HPCC

Two script files are required to run Xander. These are the configuration script and the command script. The configuration file provides paths and parameter values to be used. The command script parses the command you enter for the steps to be run. It also includes commands for several post processing steps that summarize the results.

### Configuration Script

The installed template file xander\_setenv.sh provides configuration information appropriate to the installation of RDPTools on MSU's HPCC and is divided into three sections. The first section gives the absolute paths to the sequence file, output directory, HMM files, and programs. The second section gives a short name that will be prefixed to the contig names. The third section gives parameters for each step in assembling the contigs. These parameters are specific to the data set, but usually no more than two or three need to be changed. The file is commented to help you edit it to your particular installation and data set. It is good practice to copy it to your working directory and give it a new name, such as my\_xander\_setenv.sh that will be used in this example. The installed configuration script template is:

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
    LIMIT_IN_SECS=100 # number of seconds a search allowed for each kmer, recommend 100 secs if PATHS is 1, need to increase if PATHS is large 

    ## Contig Merge Parameters
    MIN_BITS=50  # mimimum assembled contigs bit score
    MIN_LENGTH=150  # minimum assembled protein contigs

    ## Contig Clustering Parameters
    DIST_CUTOFF=0.01  # cluster at aa distance 

    NAME=k${K_SIZE}

    #### end of configuration

### Xander Command Script

The command script run\_xander\_skel.sh is printed below. 

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


## Exercise

Run Xander Interactively on MSU's HPCC using the following steps:

**1.** Create a working directory in your home directory, e.g. ~/test_xander

    mkdir ~/test_xander

**2.** Move into your working directory.

    cd ~/test_xander

**3.** Copy xander\_setenv.sh to your working directory and rename it to my\_xander\_setenv.sh.

    cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/xander_setenv.sh ~/test_xander/my_xander_setenv.sh

**4.** Edit the path variable for your working directory in the First section of my\_xander\_setenv.sh to the absolute path, e.g ~/test_xander. To start the editor, type:

    nano  my_xander_setenv.sh

Use the arrow keys to move the cursor to the appropriate place in the text. Insert characters by typing. Delete characters with the delete or backspace keys. There is a menu at the bottom of the nano screen indicating keys to use to write out the changed file and exit nano. The "^" in this menu means the "Ctrl" key. When you are finished making changes, hold down the Ctrl key and type o. You will be offered the opportunity to change the file name. To keep the same name, just hit the Enter key. If you are asked whether or not to over-write the file, type y and Enter. Then hold down the Ctrl key and type x to exit nano. You may check that your change has been made with less:

    less my_xander_setenv.sh

**5.** Copy run\_xander\_skel.sh to your working directory.

    cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/run_xander_skel.sh ~/test_xander/run_xander_skel.sh

Make sure that the two script files are executable. Change the file permissions if necessary.

    ls -l
    chmod 755 my_xander_setenv.sh
    ls -l

For this example, you do not need to edit anything in run\_xander\_skel.sh. 

**6.** The following example command will attempt to run all the three steps (build, find and search) for the genes nifH, nirK, rplB, and nosZ specified in the input parameters. It creates an assembly output directory "k45" for kmer length of 45. It makes an output directory for each gene inside "k45" and saves all the output in the gene output directories. This toy data set should take approximately 5 minutes to run. Messages will be echoed to the screen during this time. The program is finished when the input prompt reappears.
    
    ./run_xander_skel.sh my_xander_setenv.sh "build find search" "nifH nirK rplB nosZ"


You can also run the three steps separately, or search multiple genes in parallel.

    ./run_xander_skel.sh my_xander_setenv.sh "build find" "nifH nirK rplB nosZ"
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nifH" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nirK" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "rplB" &
    ./bin/run_xander_skel.sh my_xander_setenv.sh "search" "nosZ" &
    
 
**IMPORTANT:** If you want to rebuild the bloom graph structure, you need to manually delete the bloom file in the output directory. If you want to rerun  finding the starting kmers for a gene, you need to manually delete that gene output directory. As a safety precaution, the script will not automatically over-write these files, but they are specific to the data being searched in subsequent assembly (search) steps. That is, they have to be created for each data set.