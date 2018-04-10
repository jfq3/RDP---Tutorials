## Submitting Xander Jobs to MSU's HPCC
You may notice that the `Xander_assmebler/bin` directory contains the script files `qsub_run_xander_skel.sh` and `qsub_xander_setenv.sh`. Many of the parameters in `qsub_xander_setenv.sh` are commented out, and instead these parameters are supplied by the suggested qsub submission command in line 9 of `qsub_run_xander_skel.sh`. Many find this means of submitting jobs confusing, and it is unnecessary. Instead, jobs can be submitted following the example here, just as any other job is submitted.

Exercise steps are:

1. Create a working directory.
2. Copy the configuration and command scripts into the working directory.
3. Edit the configuration script.
4. Create a qsub file, named for example `submit_xander.qsub`.
5. From the working directory, enter a run command of the form:

```
qsub submit_xander.qsub
```
### Copy Scripts into Working Directory

**1.** Create a working directory in your home directory, e.g. `~/xander_qsub`

```
mkdir ~/xander_qsub
```

**2.** Move into your working directory.

```
cd ~/xander_qsub
```

**3.** Copy `xander_setenv.sh` to your working directory renaming it to `my_xander_setenv.sh` in the process.

```
cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/xander_setenv.sh ~/xander_qsub/my_xander_setenv.sh
```

**4.** Copy `run_xander_skel.sh` to your working directory, renaminig it to `run_xander.sh` in the processl.

```
cp /mnt/research/rdp/public/RDPTools/Xander_assembler/bin/run_xander_skel.sh ~/xander_qsub/run_xander.sh
```
### Edit the Configuration Script File

There are two lines in the configuration script that you need to change. With the editor of your choice, edit the path variable for your working directory in the First section of `my_xander_setenv.sh` to the absolute path, e.g `~/xander_qsub`. Edit the number of threads to use in the Third section to THREADS=3. THREADS should be 3, the number of genes you will list in the last line of the qsub file (below). After your edits, these two lines should look like this:

```
WORKDIR=~/xander_qsub

THREADS=3
```
To make these changes using the editor **nano**, begin by entering the following:

```
nano my_xander_setenv.sh
```
Use the arrow keys to move the cursor to the appropriate place in the text. Insert characters by typing. Delete characters with the delete or backspace keys. There is a menu at the bottom of the **nano** screen indicating keys to use to write out the changed file and exit **nano**. The "^" in this menu means the "Ctrl" key. When you are finished making changes, hold down the Ctrl key and type o. You will be offered the opportunity to change the file name. To keep the same name, just hit the Enter key. Then hold down the Ctrl key and type x to exit **nano**. You may check that your changes have been made with **less**:

```
less my_xander_setenv.sh
```
### Do Not Edit the Run Script File

You do not need to edit `run_xander.sh`. The path for `BASEDIR` is already set correctly for running this execise on MSU's HPCC. But do make sure that it is executable. If not make it so:

```
ls -l
chmod 755 run_xander.sh
ls -l
```
### Create the qsub File

Using the editor of your choice, create a file named `submit_xander.qsub` in your working directory. This can be done with **nano** if you wish. The contents of the file should be:

```
#!/bin/bash -login
#PBS -l walltime=00:10:00,nodes=01:ppn=4,mem=2GB
#PBS -N xander_qsub
#PBS -j oe
#PBS -m abe
#PBS -M your_email_address

export OMP_NUM_THREADS=4

cd ~/xander_qsub
./run_xander.sh my_xander_setenv.sh "build find search" "nifH nirK rplB"
```
ppn and OMP_NUM_THREADS should be one more than the number of genes, one more than the number of THREADS in `xander_setenv.sh`. And of course, replace "your_email_address" with your own actual one.

### Submit the Job

Submit the job to the queue with the command:

```
qsub submit_xander.qsub
```

