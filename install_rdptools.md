## Installing RDPTools
RDPTools must be run in  Linux-like environment, e.g. a cluster at your own institution, Mac OS X, or Ubuntu. If you are using Windows, you may install [Ubuntu](https://www.ubuntu.com/download "Ubuntu") inside [Oracle Virtual Box](https://www.virtualbox.org/ "Virtual Box") or install Ubuntu as part of a [dual boot system](https://help.ubuntu.com/community/WindowsDualBoot "Windows/Ubuntu dual boot system"). When installing Ubuntu be sure to choose an LTS (Long Term Service) version. These are normally supported for three years, while other versions are supported for only a few months. 

Processing large data sets with some RDPTools commands requires more memory than is available on notebook computers. Still, installing RDPTools on your own computer allows you to get familiar with the commands and to test scripts with smaller data sets before committing jobs to a cluster.  

### Required Programs 

RDPTools and its dependencies are available at these URLs:

* RDPTools ([https://github.com/rdpstaff/RDPTools](https://github.com/rdpstaff/RDPTools "RDPTools")) 
* Python 2.7+ ([https://www.python.org/](https://www.python.org/ "Python"))
* Java 1.6+ JDK ([https://www.oracle.com/downloads/index.html](https://www.oracle.com/downloads/index.html "Java"))
* HMMER 3.1 ([http://hmmer.janelia.org](http://hmmer.org/ "HMMER"))
* UCHIME ([http://drive5.com/uchime/uchime_download.html](http://drive5.com/uchime/uchime_download.html "UCHIME"))
* A patched version of HMMER 3.0 if building your own hidden Markov models for other genes of interest. See **Patched HMMER 3.0**  at the end of this section for instructions.

**Python 2.7** is included in most Linux systems including Ubuntu 16.04 LTS, the latest LTS version as of July 2017. Test for it by typing "python" and Enter or Return in the terminal window. Exit python by entering Control D. 

#### Preliminaries

Installation of RDPTools depends on the programs *git* and *ant*, so you may have to install them first. To test if *git* is already present, type "man -k git" followed by Enter in the terminal window. If "nothing appropriate" is returned, you will have to install *git*. Test for the presence of *ant* in the same way.

To install *ant* and *git*, enter the following in the terminal window:
    
    sudo apt-get update
    sudo apt-get install git
    sudo apt-get install ant

#### Install Java
You may test for the installation of Java by entering:

    java

which should return a list of instructions. If it does not, install Java by entering the following in the terminal window:

    sudo apt-get install default-jdk

#### Install RDPTools

Begin installation of RDPTools by cloning them from the Git repository. The commands below will install RDPTools in the directory *usr/local*:

    cd /usr/local
    git clone https://github.com/rdpstaff/RDPTools.git

Then for a new installation of RDPTools, enter:
    
    cd /usr/local/RDPTools
    git submodule init
    git submodule update
    make

Alternatively, to update an existing installation of RDPTools:

    cd /usr/local/RDPTools
    git pull
    git submodule update
    make clean
    make

Xander is one of the RDPTools. Test for the installation of Xander by entering:

    java -Xmx2g -jar /usr/local/RDPTools/hmmgs.jar

which should return a list of Xander commands.


**HMMER 3.1**

Install HMMER 3.1 by entering the following in the terminal:

    sudo apt-get install hmmer

Check that **HMMER** is installed:

    man -k hmmer

which should return a list of commands.

**UCHIME** 

Download the Linux binary [UCHIME](http://drive5.com/uchime/uchime_download.html "UCHIME4.2.40_linuxi86"). Move it to */usr/local/bin/*, renaming it to uchime in the process. As of July 2017, the UCHIME version is UCHIME4.2.40_linuxi86. Assuming you downloaded it to *~/Downloads*, the command to move and rename the file would be:

    sudo mv ~/Downloads/uchime4.2.40_i86linux32 /usr/local/bin/uchime

From the *usr/local/bin/* directory, use the sudo command to change the file permissions so that it is executable:

    cd /usr/local/bin
    sudo chmod 755 uchime


**Patched HMMER 3.0**

Installation of a patched version of **HMMER** version 3.0 is necessary only if you intend to add capability for genes not already in the Xander installation yourself. If you are not comfortable doing so, skip this installation and get help from the RDP staff.

Download **HMMER 3.0** from hmmer.org/download.html. It is the form of a compressed  file named hmmer-3.0.tar.gz.

Place the compressed file in the directory /usr/.

    sudo mv ~/Downloads/hmmer-3.0.tar.gz /usr/

Extract the file:

    cd /usr
    tar xzf hmmer-3.0.tar.gz

This will create a directory named hmmer-3.0. Rename the directory hmmer-3.0_xanderpatch.

    sudo mv hmmer-3.0 hmmer-3.0_xanderpatch

Apply the patch:

    sudo patch /usr/hmmer-3.0_xanderpatch/src/p7_prior.c < /usr/RDPTools/Xander_assembler/bin/hmmer-3.0_Xander_patch.txt

Install the patched version following the instructions in the INSTALL file. You will likely need to use sudo.

    cd /usr/hmmer-3.0_xanderpatch
    sudo ./configure
    sudo make
    sudo make install

The patched 3.0 version will be installed in */usr/local/bin/*.
If you followed the instructions above, the original 3.1 version is in /usr/bin/.
This is important to remember when using **HMMER** because commands for the two versions have the same names. The directories must be specified.







