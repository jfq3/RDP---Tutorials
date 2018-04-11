## Interactive Xander Example on MSU's HPCC

This exercise is applicable to a the installation of **RDPTools** on MSU's HPCC. Exercise steps are:

1. Create a working directory.
2. Copy the configuration and command scripts into the working directory.
3. Edit the configuration script.
4. From the working directory, enter a run command of the form:

```
./run_xander.sh my_xander_setenv.sh "build find search" "nifH nirK nosZ rplB"
```
### Copy Scripts into Working Directory

**1.** Create a working directory in your home directory, e.g. `~/test_xander`

```
mkdir ~/test_xander
```

**2.** Move into your working directory.

```
cd ~/test_xander
```

**3.** Copy `xander_setenv.sh` to your working directory renaming it to `my_xander_setenv.sh` in the process.

```
cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/xander_setenv.sh ~/test_xander/my_xander_setenv.sh
```

**4.** Copy `run_xander_skel.sh` to your working directory, renaminig it to `run_xander.sh` in the processl.

```
cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/run_xander_skel.sh ~/test_xander/run_xander.sh
```

### Edit the Configuration Script File

The configuration script is divided into three sections. The first section gives the absolute paths to the sequence (data) file, output directory, HMM files, and programs. The second section gives a short name that will be prefixed to the contig names. This name must be unique for each sample in an experiment. The third section gives parameters for each step in assembling the contigs. These parameters are specific to the data set, but usually no more than two or three need to be changed. (See the section **Choosing Xander Parameters**.) The configuraton file is commented to help you edit it for your particular installation and data set.

To run this exercise on MSU's HPCC, there is only one line in the configuration script that you need to change. With the editor of your choice, edit the path variable for your working directory in the First section of `my_xander_setenv.sh` to the absolute path, e.g `~/test_xander`. After your edits, this section should look like this:

```
## First section
## THIS SECTION MUST BE MODIFIED FOR YOUR FILE SYSTEM. MUST BE ABSOLUTE PATH
## SEQFILE can use wildcards to point to multiple files (fasta, fastq, or 
## gz format), as long as there are no spaces in the names.
SEQFILE=/mnt/research/rdp/public/RDPTools/Xander_assembler/testdata/test_reads.fa
WORKDIR=~/test_xander
REF_DIR=/mnt/research/rdp/public/RDPTools/Xander_assembler
JAR_DIR=/mnt/research/rdp/public/RDPTools
UCHIME=/mnt/research/rdp/public/thirdParty/uchime-4.2.40/uchime
HMMALIGN=/opt/software/HMMER/3.1b1--GCC-4.4.5/bin/hmmalign
```
To make this change using the editor **nano**, begin by entering the following:

```
nano my_xander_setenv.sh
```
Use the arrow keys to move the cursor to the appropriate place in the text. Insert characters by typing. Delete characters with the delete or backspace keys. There is a menu at the bottom of the **nano** screen indicating keys to use to write out the changed file and exit **nano**. The "^" in this menu means the "Ctrl" key. When you are finished making changes, hold down the Ctrl key and type o. You will be offered the opportunity to change the file name. To keep the same name, just hit the Enter key. Then hold down the Ctrl key and type x to exit **nano**. You may check that your changes have been made with **less**:

```
less my_xander_setenv.sh
```
### Do Not Edit the Run Script File

To run this example on MSU's HPCC, you do not need to edit anything in `run_xander.sh`. The path for `BASEDIR` is already set correctly.

### Run Xander

Make sure that the two script files are executable. Change the file permissions if necessary.

```
ls -l
chmod 755 my_xander_setenv.sh
chmod 755 run_xander.sh
ls -l
```

The following example command will attempt to run all three steps (build, find and search) for the genes nifH, nirK, rplB, and nosZ specified in the input parameters. It creates an assembly output directory `k45` for kmer length of 45. It makes an output directory for each gene inside the `k45` directory and saves all the output in the gene output directories. This toy data set should take approximately 5 minutes to run. Messages will be echoed to the screen during this time. The program is finished when the input prompt reappears.
    
```
./run_xander.sh my_xander_setenv.sh "build find search" "nifH nirK rplB nosZ"
```

You can also run the three steps separately, or search multiple genes in parallel.

```
./run_xander.sh my_xander_setenv.sh "build find" "nifH nirK rplB nosZ"
./run_xander.sh my_xander_setenv.sh "search" "nifH" &
./run_xander.sh my_xander_setenv.sh "search" "nirK" &
./run_xande.sh my_xander_setenv.sh "search" "rplB" &
./bin/run_xander.sh my_xander_setenv.sh "search" "nosZ" &
```

**IMPORTANT:** If you want to rebuild the bloom graph structure, you need to manually delete the bloom file (`k45.bloom`) in the output directory. If you want to rerun  finding the starting kmers for a gene, you need to manually delete that gene's output directory (in this case under `k45`). As a safety precaution, the script will not automatically over-write these files, but they are specific to the data being searched in subsequent assembly (search) steps. That is, they have to be created for each data set.

