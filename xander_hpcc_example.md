## Interactive Xander Example on MSU's HPCC

Two script files are required to run Xander. These are the configuration script (`xander_setenv.sh`) and the command script (`run_xander_skel.sh`). 
	* `xander_setenv.sh`: The configuration file provides paths and parameter values to be used. This __must__ be adjusted before running. See [xander_choosing_parameters.md](https://github.com/dunivint/RDP_Tutorials/blob/master/xander_choosing_parameters.md) for more information.
	* `run_xander_skel.sh`: The command script parses the command you enter for the steps to be run. It also includes commands for several post processing steps that summarize the results. This does not need to be edited. 

### Configuration Script (`xander_setenv.sh`)

The installed template file `xander_setenv.sh` provides configuration information appropriate to the installation of RDPTools on MSU's HPCC and is divided into three sections. 
	* The first section gives the absolute paths to the sequence file, output directory, HMM files, and programs. 
	* The second section gives a short name that will be prefixed to the contig names. 
	* The third section gives parameters for each step in assembling the contigs. These parameters are specific to the data set, but usually no more than two or three need to be changed. 
	
This script is located within the Xander package: `RDPTools/Xander_assembler/bin/`

It is good practice to copy this script to your working directory and give it a new name, such as `my_xander_setenv.sh` that will be used in this example. The installed configuration script template is:

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

### Xander Command Script (`run_xander_skel.sh`)

The command script `run_xander_skel.sh` is also included with Xander (`RDPTools/Xander_assembler/bin/`) and is printed below. 

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

**3.** Copy `xander_setenv.sh` to your working directory and rename it to `my_xander_setenv.sh`.

    cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/xander_setenv.sh ~/test_xander/my_xander_setenv.sh

**4.** Edit the path variable for your working directory in the First section of `my_xander_setenv.sh` to the absolute path, e.g ~/test_xander. To start the editor, type:

    nano  my_xander_setenv.sh

* Nano tips: Use the arrow keys to move the cursor to the appropriate place in the text. Insert characters by typing. Delete characters with the delete or backspace keys. There is a menu at the bottom of the nano screen indicating keys to use to write out the changed file and exit nano. The "^" in this menu means the "Ctrl" key. When you are finished making changes, hold down the Ctrl key and type o. You will be offered the opportunity to change the file name. To keep the same name, just hit the Enter key. If you are asked whether or not to over-write the file, type y and Enter. Then hold down the Ctrl key and type x to exit nano. 

* You may check that your change has been made with less:

    less my_xander_setenv.sh

**5.** Copy `run_xander_skel.sh` to your working directory.

    cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/run_xander_skel.sh ~/test_xander/run_xander_skel.sh

Make sure that the two script files are executable. Change the file permissions if necessary.

    ls -l
    chmod 755 my_xander_setenv.sh
    ls -l

For this example (and in most other cases), you do not need to edit anything in `run_xander_skel.sh`. 

**6.** The following example command will attempt to run all the three steps (build, find and search) for the genes nifH, nirK, rplB, and nosZ specified in the input parameters. It creates an assembly output directory "k45" for kmer length of 45. It makes an output directory for each gene inside "k45" and saves all the output in the gene output directories. This toy data set should take approximately 5 minutes to run. Messages will be echoed to the screen during this time. The program is finished when the input prompt reappears.
    
    ./run_xander_skel.sh my_xander_setenv.sh "build find search" "nifH nirK rplB nosZ"


You can also run the three steps separately, or search multiple genes in parallel.

    ./run_xander_skel.sh my_xander_setenv.sh "build find" "nifH nirK rplB nosZ"
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nifH" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nirK" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "rplB" &
    ./bin/run_xander_skel.sh my_xander_setenv.sh "search" "nosZ" &
    
 
**IMPORTANT:** If you want to rebuild the bloom graph structure, you need to manually delete the bloom file in the output directory. If you want to rerun  finding the starting kmers for a gene, you need to manually delete that gene output directory. As a safety precaution, the script will not automatically over-write these files, but they are specific to the data being searched in subsequent assembly (search) steps. That is, they have to be created for each data set.

## Submitting Xander Jobs to MSU's HPCC

Submitting Xander jobs to MSU's HPCC is somewhat different from submitting other jobs. Because of the way Xander is written, a qsub file is not used. Instead, requests for resources are given following the -l flag in the qsub command, and  several Xander parameters are given following the -v flag. Wherever required, full paths should be given. An example of a valid qsub command is included near the beginning of the configuration script. Parameters included in the qsub command do not need to be included in the configuration file. If they are present in the configuration file they should agree with the parameters in the qsub command. For the following exercise they have been commented out.

### Exercise

1. Log on to the HPCC and create the directory *xander_qsub* in your home directory. 
1. Put the configuration and run scripts below in the *xander_qsub* directory. Edit occurrences of <your_id> to, you guessed it, your MSU ID. Make sure the files contain unix line endings. Save them as *qsub_xander_run.sh* and *qsub_xander_setenv.sh*. Make sure they are readable and that *qsub_xander_run.sh* is executable.
1. Edit occurrences of <your_id> in the qsub comamnd to your MSU id.
1. Submit the qsub command from the *xander_qsub* directory.

### Model qsub command

    qsub -l walltime=0:30:00,nodes=01:ppn=2,mem=2GB -v MAX_JVM_HEAP=2G,FILTER_SIZE=32,K_SIZE=45,MIN_COUNT=2,tasks="build find search",genes="nifH",THREADS=1,SAMPLE_SHORTNAME=test,WORKDIR=/mnt/home/<your_id>/xander_qsub,ENVFILE=/mnt/home/<your_id>/xander_qsub/qsub_xander_setenv.sh,SEQFILE=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata/test_reads.fa qsub_xander_run.sh

### Qsub configuration file

Here is the accompanying configuration file *qsub_xander_setenv.sh*. Parameters given in the qsub command have been commented out. These include the WORKDIR, given here as */mnt/home/<your_id>/xander_qsub/*. It is good practice to edit this to your ID even though it is not used, and to make sure that all parameters given in the qsub command agree with the values here in the configuration file. That way you have a record of what you did.

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
    # SEQFILE=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata/test_reads.fa
    # WORKDIR=/mnt/home/<your_id>/xander_qsub/
    REF_DIR=/mnt/research/rdp/public/RDPTools/Xander_assembler/
    JAR_DIR=/mnt/research/rdp/public/RDPTools/
    UCHIME=/mnt/research/rdp/public/thirdParty/uchime-4.2.40/uchime
    HMMALIGN=/opt/software/HMMER/3.1b1--GCC-4.4.5/bin/hmmalign
    
    
    # THIS SECTION NEED TO BE MODIFIED FOR GENES OF INTEREST, and SAMPLE_SHORTNAME WILL BE THE PREFIX OF CONTIG ID
    # genes=(nifH)
    # SAMPLE_SHORTNAME=test
    
    # THIS SECTION MUST BE MODIFIED BASED ON THE INPUT DATASETS
    # De Bruijn Graph Build Parameters
    # K_SIZE=45  # kmer size, should be multiple of 3
    # FILTER_SIZE=32 # memory = 2**FILTER_SIZE, 38 = 32 GB, 37 = 16 GB, 36 = 8 GB, 35 = 4 GB, increase FILTER_SIZE if the bloom filter predicted false positive rate is greater than 1%
    # MAX_JVM_HEAP=2G # memory for java program, must be larger than the corresponding memory of the FILTER_SIZE
    # MIN_COUNT=2  # minimum kmer abundance in SEQFILE to be included in the final de Bruijn graph structure
    
    # ppn should be THREADS +1
    # THREADS=1
    
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

### Qsub Run File
    
Here is the accompanying run file *qsub_xander_run.sh*:

    #!/bin/bash -login
    
    BASEDIR=/mnt/research/rdp/public/RDPTools/Xander_assembler/bin/
    
    #### start of configuration
    source ${ENVFILE}
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

    
