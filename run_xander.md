## Running Xander
Two script files are required to run Xander. These are the configuration script (`xander_setenv.sh`) and the command script (`run_xanderl.sh`). A template version of `run_xander.sh` is supplied as `run_xander_skel.sh` in the `Xander/bin` directory. It is to be copied and edited as described in the three sections below, depending on how Xander is being run.
* `xander_setenv.sh`: This configuration file provides paths and parameter values to be used. This **must** be edited before running Xander. See the examples below and the section **Choosing Xander Parameters** for more information.
* `run_xander.sh`: This command script parses the command you enter for the steps to be run. It also includes commands for several post processing steps that summarize the results. One line in this file needs to be edited if you are not using MSU's HPCC to run Xander. 
The general command to run Xander from a working directory containing both of the above files is:

```
./run_xander.sh xander_setenv.sh "build find search" "nifH nirK rplB"
```
The steps to run are enclosed in the first set of parentheses and the genes to be found are enclosed in the second set of parentheses.

### Suggestions for Running Xander
The build step builds the bloom filter. It must be run for each data set. For real data sets, it is suggested that the build step be run separately from the other steps, and the log file examined to make sure that the error rate is below 1 percent. The build step is not currently mult-threaded, so there is no advantage in requesting multiple threads when it is run.

Once the bloom filter is built, it may be used to assemble multiple genes. The find step finds starting kmers for each gene. The search step assembles the gene sequences and performs some further processing. These steps may be run separately, but the find step must be run before the search step. These steps are multi-threaded.