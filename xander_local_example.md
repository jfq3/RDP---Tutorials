## Test Your Local Xander Installation

Two script files are required to run Xander. These are the configuration script and the command script. The configuration file provides paths and parameter values to be used. The command script parses the command you enter for the steps to be run. It also includes commands for several post processing steps that summarize the results.

Template files for these two script files are part of the RDPTools installation and are located in RDPTools/Xander\_assembler/bin sub-directory. The configuration script file is named xander\_setenv.sh and the command script file is named run\_xander\_skel.sh. These files are appropriate to the installation of RDPTools on MSU's HPCC. They must be edited for use on another installation of RDPTools.


### Edit the Configuration Script

The configuration script is divided into three sections. The first section gives the absolute paths to the sequence file, output directory, HMM files, and programs. The second section gives a short name that will be prefixed to the contig names. The third section gives parameters for each step in assembling the contigs. These parameters are specific to the data set, but usually no more than two or three need to be changed. The file is commented to help you edit it to your particular installation and data set. It is good practice to copy it to your working directory and give it a new name, such as my\_xander\_setenv.sh that will be used in this example.

If you followed the instructions for installing RDPTools in directory /usr, then enter commands below in the terminal. If you installed RDPToools in some other directory, you will have to modify the paths in the commands to point to RDPTools.

    cd ~
    mkdir test_xander
    cd test_xander
    cp /usr/RDPTools/Xander_assembler/bin/xander_setenv.sh ~/test_xander/my_xander_setenv.sh
    cp /usr/RDPTools/Xander_assembler/bin/run_xander_skel.sh ~/test_xander/run_xander_skel.sh

Next, begin editing the paths in the first section of my\_xander\_setenv.sh by entering the following:

    nano my_xander_setenv.sh

Use the arrow keys to move the cursor to the appropriate place in the text. Insert characters by typing. Delete characters with the delete or backspace keys. There is a menu at the bottom of the nano screen indicating keys to use to write out the changed file and exit nano. The "^" in this menu means the "Ctrl" key. When you are finished making changes, hold down the Ctrl key and type o. You will be offered the opportunity to change the file name. To keep the same name, just hit the Enter key. Then hold down the Ctrl key and type x to exit nano. You may check that your changes have been made with less:

    less my\_xander\_setenv.sh

If you installed RDPTools in directory /usr and followed the other instructions for installing uchime and hmmer, the edited section should look like the below, but be sure that all paths and file names match your installations.

    ## THIS SECTION MUST BE MODIFIED FOR YOUR FILE SYSTEM. MUST BE ABSOLUTE PATH
    ## SEQFILE can use wildcards to point to multiple files (fasta, fastq or gz format), as long as there are no spaces in the names
    SEQFILE=/usr/RDPTools/Xander_assembler/testdata/test_reads.fa
    WORKDIR=~/test_xander
    REF_DIR=/usr/RDPTools/Xander_assembler/
    JAR_DIR=/usr/RDPTools/
    UCHIME=/usr/uchime/uchime4.2.40_i86linux32
    HMMALIGN=/usr/bin/hmmalign

To process the test\_reads.fa provided, you do not need to edit sections 2 or 3 of the configuration file.

### Edit the Command Script File

Use nano as above to edit the value of BASEDIR in run\_xander\_skel.sh to point to the RDPTools/Xander_assembler/bin directory. If you installed RDPTools in the directory /bin, this line should look like this:

    BASEDIR=/usr/RDPTools/Xander_assembler/bin

Make sure that the two script files are executable. Change the file permissions if necessary.

    ls -l
    chmod 755 my_xander_setenv.sh
    ls -l

### Run Xander

The following example command will attempt to run all the three steps (build, find and search) for the genes nifH, nirK, rplB, and nosZ specified in the input parameters. It creates an assembly output directory "k45" for kmer length of 45. It makes an output directory for each gene inside "k45" and saves all the output in the gene output directories. This toy data set should take approximately 5 minutes to run. Messages will be echoed to the screen during this time. The program is finished when the input prompt reappears.
    
    ./run_xander_skel.sh my_xander_setenv.sh "build find search" "nifH nirK rplB nosZ"


You can also run the three steps separately, or search multiple genes in parallel.

    ./run_xander_skel.sh my_xander_setenv.sh "build find" "nifH nirK rplB nosZ"
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nifH" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "nirK" &
    ./run_xander_skel.sh my_xander_setenv.sh "search" "rplB" &
    ./bin/run_xander_skel.sh my_xander_setenv.sh "search" "nosZ" &
    
 
**IMPORTANT:** If you want to rebuild the bloom graph structure, you need to manually delete the bloom file in the output directory. If you want to rerun  finding the starting kmers for a gene, you need to manually delete that gene output directory. As a safety precaution, the script will not automatically over-write these files, but they are specific to the data being searched in subsequent assembly (search) steps. That is, they have to be created for each data set.