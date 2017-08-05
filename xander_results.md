## Explanation of Xander Results 

The *run\_xander\_skel.sh* script first builds the bloom filter, finds starting kmers, and assembles contigs into target gene sequences. Once the contigs have been assembled, they are corrected for insertions and deletions by **FrameBot**, clustered at 1% aa distance, and the longest contig from each cluster taken as the representative sequence for the cluster. These are filtered for chimeras using **UCHIME** to give the final sequence files ending with *\_final\_nucl.fasta*, *\_final\_prot.fasta*, and *\_final\_prot\_aligned.fasta*. (For example *test\_nifH\_45\_final\_prot\_aligned.fasta*.) The closest match to each sequence is found with **FrameBot** and a coverage file is generated. The coverage file is used to adjust the sequence counts for coverage, and a taxon abundance file (ending with *taxonabundance.txt*) based on these corrected counts is generated. This file is the final Xander result for a single sample and gives the percentage of corrected counts for each phylum or class for which sequences were found. 

The inputs and outputs for each step in this process are explained below. Input files may be from Xander's *gene_resource* directory. In many cases only the endings of the output file names are given; they may be prepended with some combination of any of the sample short name, the kmer length used, and the gene name. The output files from the build step are found in the output directory under a sub-directory named for the kmer length (e.g. *k45*). The other files are found in a sub-directory of this named for the gene or in a sub-directory named *cluster* under the gene directory.

### Build
Build the de Bruijn graph. Do this only once for each data set for a given kmer length.

* Input: read files (fasta, fastq or gz format)
* Output 1: de Bruijn graph (*k45.bloom*) 
* Output 2: bloom file stats (*k45\_bloom\_stat.txt*). Check the "Predicted false positive rate" in this file (see *xander\_choosing\_parameters.md*).

### Find
Identify the starting kmers. Multiple genes should be run together to save time; there is a multi-thread option.

* Input 1: *ref_aligned.faa* files from gene ref directories  
* Input 2: read files
* Output: starting nucleotide kmers (*starts.txt* for each gene)

### Search
Assemble the contigs. Each gene can be done in parallel. Length cutoff or HMM score cutoff filters are used.

* Input 1: forward and reverse HMMs (*for_enone.hmm* and *rev_enone.hmm*)
* Input 2: de Bruijn graph (*k45.bloom*)
* Input 3: starting kmers (*gene_starts.txt*)
* Output 1: unique merged protein contigs (*prot_merged_rmdup.fasta*)
* Output 2: merged nucleotide contigs (*nucl_merged.fasta*)
* Output 3: unmerged nucleotide and protein contigs (*gene_starts.txt_nucl.fasta* and *gene_starts.txt_prot.fasta*)

### Post Assembly Steps

The script *run\_xander\_skel.sh* includes commands for several post assembly steps as explained below.

#### Cluster
RDP's mcClust (https://github.com/rdpstaff/Clustering) is used to cluster the sequences. For each of the clusters the longest contig is chosen as the representative contig.

* Input 1: *prot_merged_rmdup.fasta*
* Output 1: representative contigs at 99% aa identity (*nucl_rep_seqs.fasta* and *prot_rep_seqs.fasta*)
* Output 2: aligned protein contigs (*aligned.fasta*)
* Output 3: complete linkage cluster output (*complete.clust*) 

#### Remove Chimeric Contigs

**UCHIME** in reference mode is used to remove chimeric contigs. The sequences in the gene reference directory are used as the database. 

* Input 1: representative nucleotide contigs (*nucl_rep_seqs.fasta*)
* Input 2: gene nucleotide reference set (*originaldata/nucl.fa* from the gene reference directory)
* Output 1: **UCHIME** output (*result_uchimealn.txt*, *results.uchime.txt*)
* Output 2: quality_filtered nucleotide representative contigs (*final_nucl.fasta*)
* Output 3: quality_filtered protein representative contigs (*final_prot.fasta*) 

**Note: the quality-filtered contigs in *final\_nucl.fasta*, *final\_prot.fasta* and *final\_prot\_aligned.fasta* should be used as the final set of contigs assembled by Xander.**

#### Find Closest Matches

The nearest reference sequence match to each contig is found using RDP's **FrameBot** tool (https://github.com/rdpstaff/Framebot). RDP's **Protein Seqmatch** tool could also be used for this step.

* Input 1: quality\_filtered nucleotide representative contigs (*final\_nucl.fasta*)
* Input 2: gene protein reference set (*originaldata/framebot.fa* from gene reference directory)
* Outputs: the nearest reference seq and % aa identity (*framebot.txt*)

#### Coverage & Kmer Abundance

Read mapping, contig coverage, and kmer abundance are determined with RDP's **KmerFilter** tool. There is a multi-thread option for these steps.

* Input 1: quality\_filtered nucleotide representative contigs (*final\_nucl.fasta*)
* Input 2: read files
* Output 1: contig coverage (*coverage.txt*) This file can be used to estimate gene abundance and adjust sequence abundance.
* Output 2: kmer abundance (*abundance.txt*)

#### Taxonomic Abundance 

 * Input 1: contig coverage (*coverage.txt*)
 * Input 2: the nearest reference seq (*framebot.txt*) 
 * Input 3: gene protein reference set (*originaldata/framebot.fa* from gene reference directory) 
 * Output: taxonomic abundance adjusted by coverage, grouped by lineage (phylum and in some cases class) (*taxonabund.txt*)

This last file is the final output for a single sample. You can think of it as a summary for OTUs (clusters) found in the sample. For each, it gives the closest reference match, the lineage of the match, and the abundance and fractional abundance. The file also summarizes this information by phylum or class.

## Multiple Samples

To see how results for multiple samples can be combined into an OTU table for community analyses, see the file *xander\_combining\_samples.md*.