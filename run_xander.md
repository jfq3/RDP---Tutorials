## Running Xander

Two script files are required to run Xander. These are the configuration script and the command script. The configuration file provides paths and parameter values to be used. The command script parses the command you enter for the steps to be run and the genes to be found. It also includes commands for several post processing steps that summarize the results.

Templates for these two script files are part of the RDPTools installation and are located in `RDPTools/Xander\_assembler/bin` sub-directory. The configuration script file is named `xander\_setenv.sh` and the command script file is named `run\_xander\_skel.sh`. These paths in these files are appropriate to the installation of RDPTools on MSU's HPCC. The paths must be edited for use on another installations of RDPTools.

### The Xander Exercises

There are three exercises in this section. The first two are run interactively, locally or on MSU's HPCC, and the third demonstrates how to submit Xander jobs to the HPCC. Gain experience editing the run and configuration files by doing one (or both) of the interactive exercises. Analysis of the toy data set included with Xander takes only a few minutes to run, so it is permissible to run interactively on the HPCC. Analysis of real data takes long enough that such jobs must be submitted to the queueing system.

