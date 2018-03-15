## Configure gene resources

Go the the FunGene page (http://fungene.cme.msu.edu/) and see if your gene of interest has been already been entered. If not, you must select seed sequences covering the diversity of your gene. It requires some expertise to do this properly. Then contact the RDP staff (
http://rdp.cme.msu.edu/misc/contacts.jsp) and request that they add your seed sequences to FunGene. FunGene searches for related sequences and updates monthly, so if your gene is not already listed it is important to submit seed sequences as soon as possible. 

Reference sequence files and models for each gene targeted for assembly are placed in a gene reference directory. This directory is usually inside the `Xander_assembler/gene_resource directory`, but may be elsewhere if necessary (e.g. in your home directory on a cluster). The analysis pipeline is preconfigured for the genes rplB, amoA_AOA, AmoA_AOB, nifH, nirK, nirS, norB_cNor, norB_qNor, nosZ, and nosZ_a2. As you can see from these names, it is sometimes necessary to divide a gene into groups, depending on its sequence diversity.

### File organization
The `gene_resource` directory is named for the gene and contains files and the subdirectory `originaldata` in this configuration:

![directory structure](gene_resource_directory_small.png)

In this example, the gene directory is `rplB`. It contains three files and the sub-directory `originaldata` which in turn contains the four files required for preparing HMMs and for post-assembly processing. These are:

- `rplB.seeds`: a small set of protein sequences in FASTA format, used to build forward and reverse HMMs for the gene. These can be downloaded from FunGene.
- `rplB.hmm`: the HMM built from rplB.seeds using **HMMER** and can also be downloaded from FunGene.
- `framebot.fa`: a large near full length known protein set for identifying starting kmers and for FrameBot nearest matching.
- `nucl.fa`: a large near full length known set used by **UCHIME** for chimera checking.

All four of these files can be downloaded from FunGene. There are links to the `gene.seeds` and `gene.hmm` files in the upper left of each FunGene gene page. Near full-length and well annotated sequences are preferred for `framebot.fa`. Download these from FunGene, filtering with minimum HMM coverage of at least 80%. More diversity is better. More sequences mean more starting points, requiring more computational time, but make the process of model creation less susceptible to noise. Download nucleotide sequences to nucl.fa in a similar manner.

### Prepare gene references

The script `prepare_gene_ref.sh`, present in `/RDPTools/Xander_assembler/bin/`, builds HMMs for Xander and aligns reference sequences. The script does this using  hmmer-3.0_xanderpatch, a modified version of HMMMER3.0. The modified version is tuned to detect close orthologs. 
* __Inputs__ include three files from the `originaldata` directory:
    * `gene.seeds`
    * `gene.hmm`
    * `framebot.fa` 
* __Outputs__ are saved to the `gene` directory within `gene_resource`
    * `for_enone.hmm`: forward HMM for assembling gene contigs
    * `rev_enone.hmm`: reverse HMM for assembling gene contigs
    * `ref_aligned.faa`: contains the framebot.fa sequences aligned with for_enone.hmm and is used by Xander to identify starting kmers.        
        * It is important to manually examine the alignment of ref_aligned.faa using Jalview or  another alignment viewing tool to spot any badly aligned sequences. If found, it is likely that there are no sequences in gene.seeds close these sequences. You need to determine if these problem sequences are actually from your gene of interest, and then either remove them or add some representative sequences to gene.seeds and repeat the preparation steps. In some cases it will not be possible to get good alignments of ref_aligned.faa without dividing  the seed sequences and framebot.fa into subsets. And of course, in such cases, the gene.hmm needs to be built for each subset of the seeds.

### Test gene references

Check the file permissions for the files in the gene resource directory to be sure they are all readable by user, group, and world. If not, make them so (`chmod 644 filename`). Then test that your new gene resource files work by running Xander with a subset of `framebot.fa` as the sample sequences (`SEQFILE`).

*insert troubleshooting ideas if xander does not successfully assemble gene resource sequences*

### Add Taxonomy to framebot.fa

Once assured that your new gene resource files work, you can add taxonomy to the `framebot.fa` file. If taxonomy is added, Xander will automatically calculate the relative abundance of each phylum. 

* Requirements
    * python *version*
    * Biopython *version*
    * Internet connection. 
    
* (If necessary) Biopython can be installed with the following command:

    `sudo apt-get install python-biopython`

* __Workflow__
    * Create a directory and copy `framebot.fa` to it. 
    * For example, if you have been working on adding capability for the gene but, you might do the following:

    mkdir ~/add_taxonomy
    cd ~/add_taxonomy
    cp /usr/RDPTools/Xander_assembler/gene_resource/but/originaldata/framebot.fa ~/add_taxonomy/

    * Make a script `SCRIPT_NAME` with the following code:

```
    #!/bin/bash
    
    # Configure the path to the pythonscripts directory:
    scripts_dir=/usr/RDPTools/Xander_assembler/pythonscripts
    
    # Parse the accession numbers to the file accnos.txt:
    cat framebot.fa | grep ">" |  sed 's/\(>\)\([0-9]*\)\(.*\)/\2/' > accnos.txt
    
    # For each accession number, download the GenBank file in xml format:
    python $scripts_dir/fetch_ncbi_xml.py protein accnos.txt ./temp xml
    
    # Parse the phylogenies from the xml files and write them to the 
    # framebot.fa sequence descriptions:
    python $scripts_dir/parse_ncbi_lineage.py  framebot.fa  ./temp  ./temp/framebot.fa
```

   * Edit the script so that the path to the `pythonscripts` directory matches your path
   * Make sure the script is executable. If not, run `chmod 744 SCRIPT` to make it executable
   * Change to the `add_taxonomy` directory
   * Then, from the `add_taxonomy` directory, run the `NAME` script below.

   * __Outputs__
        * Directory `temp` under your working directory (`~/add_taxonomy`) 
        * `framebot.fa` with taxonomy
        
   * Replace the `framebot.fa` in the `gene_resource` directory with this modified version, then Xander will be able to include taxonomy in its output.
