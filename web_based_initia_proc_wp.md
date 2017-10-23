# Introduction
The first step in processing sequencing data is to sort the sequences by sample. It is usually appropriate to trim them of primers and bar codes and filter them for quality at the same time.  These tasks may be accomplished with either web-based or command line tools.

# Objectives
To learn how to use RDP's web-based Initial Processing tool to do the following:

  * Process single 16S reads
  * Process single 28S reads
  * Process paired reads with Pandaseq

# Preparation
Windows users should have the following programs installed:  

  * A compression utility such as _WinZip_ or _Zip-7_
  * _Notepad++_
  * _PuTTY_
   
Mac users should have *TextWrangler* installed.   

Create a directory on your computer for these tutorial exercises.  I suggest that you create sub-directories for each task.  For example, for now create a tree of nested sub-directories as in the figure below:  

# Single Reads with Primers & Barcodes

## Web-based Initial Processing - 16S

Download the file `init_proc.zip` from here and place it in the sub-directory `Single_reads` you created above.  Unzip the file.  

Go to RDP's home page at http://rdp.cme.msu.edu/ and do the following:  

  * Click on the RDPipeline tile.
  * Click on "Pipeline Initial Process.""
  * Log in to your personal RDP work space, or create an account if you do not already have one.  

Some explanation of the tool is given at the top of the screen. Fill in the form with the following:  

  * A job name so you may easily identify the result later, e.g. `init_web_16S`.
  * For the sequence file, choose `example_454Reads.fastq` from the files you downloaded for this tutorial. It consists of bacterial 16S amplicons from a pyrosequencing experiment. You can compress it first if you wish.  Compressed files are accepted and reduce upload times.
  * For the tag file, choose `tag_file.txt` from the files you downloaded. 
  * For gene name, choose bacterial 16S.
  * Enter the forward and reverse primers in the appropriate boxes. These are:  
        Forward: AYTGGGYDTAAAGNG  
        Reverse: CCGTCAATTCMTTTRAGT  
      
You may copy and paste the primer sequences from the text file you downloaded (`primer.txt`).  

You set filter values at the bottom of the page. They are pre-filled with default values. Change some of them as follows:  

  * Reverse primer max edit distance to 0
  * Min sequence length to 300
  * Max sequence length to 350  

Click on "Perform Initial Processing." After the files upload, the page will display information about the job. When the job completes, you may download the results by clicking on "my jobs" at the top of the screen. Your jobs will be listed, and under the download column you will be able to download the result as a `tar/gz` file or `zip` file into your tutorial directory. Unzipping the file will create a sub-directory named `initial_process` with the results for each sample in a separate sub-directory. Or you may selectively unzip only the trimmed sample fastq files if you wish, putting them all into the same directory. I suggest that you create a directory `trimmed_seqs` under `Single_reads` and place them there.  In any event, it is convenient to zip all trimmed sample fastq files for your experiment together before proceeding to subsequent steps.   

## Web-based Initial Processing - 28S  

The primers used for amplifying fungal 28S sequences were too far apart to get sequences that included both primers, so identical bar codes were put on both primers.  This way sequences obtained from each direction could be sorted to the same sample. For initial processing, both primers were included in the forward primer box.  The sequences obtained in this way could not be aligned and clustered, but they could still be classified.  

Process example 28S sequences you received in file `28S_init_proc.zip`.  Make sure to select the gene as fungal 28S and put both primers in the forward box.  Set the filter parameters as follow:  

  * Forward primer max edit distance to 1
  * Reverse primer max edit distance to 1
  * Minimum read Q score to 20
  * Min sequence length to 200
  * Max sequence length to 400 
  
The results will be classified later and imported into phyloseq with the `hier2phyloseq` function in package `RDPutils` to give an object with both OTU and classification tables.  

# Web-based Initial Processing of Paired Reads

MiSeq results for paired reads are usually returned as sorted pairs (representing samples) of fastq files already trimmed of primers and bar codes.  In this case it is only necessary to assemble the paired reads into sequences.  If they do still contain primers and bar codes, they can be sorted by sample and trimmed for length and quality the same as for single reads above.  Just provide a tag file, primer sequences, and filter parameters as appropriate.  

Download the file `mock_miseq_16s.tgz` from here. It contains two pairs of reads.  You may upload it as is.  You do not need to provide a tag file or primer sequences, but be sure to choose Bacterial 16S as the gene name.  

Set the filter parameters as follow:  

  * Minimum read Q score to 25
  * Min sequence length to 220
  * Max sequence length to 280 
  
Check the small box for "Assemble paired end reads" and click "Perform Processing."  Download and unzip your results.  The assembled reads and log files will be in the folder `assembled_paired_end_sequences`.  

