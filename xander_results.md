# Explanation of Xander Results 

The *run\_xander\_skel.sh* script first builds the bloom filter, finds starting kmers, and assembles contigs into target gene sequences. Once the contigs have been assembled, they are corrected for insertions and deletions by **FrameBot**, clustered at a specified aa distance, and the longest contig from each cluster taken as the representative sequence for the cluster. These are filtered for chimeras using **UCHIME** to give the final sequence files ending with *\_final\_nucl.fasta*, *\_final\_prot.fasta*, and *\_final\_prot\_aligned.fasta*. (For example *test\_nifH\_45\_final\_prot\_aligned.fasta*.) The closest match to each sequence is found with **FrameBot** and a coverage file is generated. The coverage file is used to adjust the sequence counts for coverage, and a taxon abundance file (ending with *taxonabundance.txt*) based on these corrected counts is generated. This file is the final Xander result for a single sample and gives the percentage of corrected counts for each phylum or class for which sequences were found. 

The inputs and outputs for each step in this process are explained below. Input files may be from Xander's *gene_resource* directory. In many cases only the endings of the output file names are given; they may be prepended with some combination of any of the sample short name, the kmer length used, and the gene name. The output files from the build step are found in the output directory under a sub-directory named for the kmer length (e.g. *k45*). The other files are found in a sub-directory of this named for the gene or in a sub-directory named *cluster* under the gene directory.

---

## Build
Build the de Bruijn graph. Do this only once for each data set for a given kmer length.

* Input: read files (fasta, fastq or gz format)

* Output
   * de Bruijn graph (`k45.bloom`) 
   * bloom file stats (`k45_bloom_stat.txt`). Check the "Predicted false positive rate" in this file to make sure that it is less than 1% ( i.e. < 0.01, see **Choosing Xander Parameters**.)

---

## Find
Identify the starting kmers. Multiple genes should be run together to save time; there is a multi-thread option.

* Input
   * `ref_aligned.faa` files from `gene_resource` directory 
   * read files (fasta, fastq or gz format)
   
* Output: starting nucleotide kmers (`starts.txt` for each gene)

---

## Search: Initial search
Assemble the contigs. Each gene can be done in parallel. Length cutoff or HMM score cutoff filters are used. Caution: no outputs from this step are quality filtered!

* Input
   * forward and reverse HMMs (`for_enone.hmm` and `rev_enone.hmm`)
   * de Bruijn graph (`k45.bloom`)
   * starting kmers (`gene_starts.txt`)
   
* Output
   * unique merged protein contigs (`prot_merged_rmdup.fasta`)
   * merged nucleotide contigs (`nucl_merged.fasta`)
   * unmerged nucleotide and protein contigs (`gene_starts.txt_nucl.fasta` and `gene_starts.txt_prot.fasta`)

---

## Search: Post assembly processing
Post assembly steps including clustering, chimera removal, closest-match assignment, and abundance calculation.

### Cluster
RDP's [mcClust](https://github.com/rdpstaff/Clustering) is used to cluster the sequences based on aa identity. For each of the clusters the longest contig is chosen as the representative contig. Caution: no outputs from this step are quality filtered! All outputs for this step are located in the `cluster` directory. 

* Input: `prot_merged_rmdup.fasta` from initial search

* Output
   * representative contigs at 99% aa identity (`nucl_rep_seqs.fasta` and `prot_rep_seqs.fasta`). 
   * aligned protein contigs (`aligned.fasta`)
   * complete linkage cluster output (`complete.clust`). Shows how many contigs you have with different distance cutoffs.

---

### Remove Chimeric Contigs
**UCHIME** in reference mode is used to remove chimeric contigs. All outputs for this step are located in the `cluster` directory. 

* Input
   * representative nucleotide contigs (`nucl_rep_seqs.fasta` from cluster step)
   * gene nucleotide reference set ( `nucl.fa` from the `gene_reference/GENE/originaldata` directory)
   
* Output
   * **UCHIME** output (`result_uchimealn.txt`, `results.uchime.txt`)
   * quality_filtered nucleotide representative contigs (`final_nucl.fasta`)
   * quality_filtered protein representative contigs and their raw abundance (not coverage adjusted; number of contigs)(`final_prot.fasta`) 

**Note: the quality-filtered contigs in `final_nucl.fasta`, `final_prot.fasta` and `final_prot_aligned.fasta` should be used as the final set of contigs assembled by Xander.**

---

### Find Closest Matches
The nearest reference sequence match to each contig is found using RDP's [**FrameBot** tool](https://github.com/rdpstaff/Framebot). RDP's [**Protein Seqmatch** tool](https://github.com/rdpstaff/SequenceMatch) could also be used for this step. All outputs for this step are located in the `cluster` directory. 

* Input
   * quality_filtered nucleotide representative contigs (`final_nucl.fasta`)
   * gene protein reference set (`framebot.fa` from the `gene_reference/GENE/originaldata` directory)
   
* Output: the nearest reference seq and % aa identity (`framebot.txt`)

---

### Coverage & Kmer Abundance
Read mapping, contig coverage, and kmer abundance are determined with RDP's [**KmerFilter** tool](https://github.com/rdpstaff/KmerFilter). There is a multi-thread option for these steps.

* Input 1
   * quality\_filtered nucleotide representative contigs (`final_nucl.fasta`)
   * read files (fasta, fastq or gz format)
* Output
   * contig coverage (`coverage.txt`) This file can be used to estimate gene abundance and adjust sequence abundance.
   * kmer abundance and corresponding frequency (`abundance.txt`)

---

### Taxonomic Abundance 
This gives is the final output for a single sample. You can think of it as a summary for OTUs (clusters) found in the sample. For each, it gives the closest reference match and the abundance and fractional abundance. 

 * Input
   * contig coverage (`coverage.txt`)
   * the nearest reference seq (`framebot.txt`) 
   * gene protein reference set (`framebot.fa` from the `gene_reference/GENE/originaldata` directory) 
   
 * Output: taxonomic abundance adjusted by coverage, grouped by lineage (phylum and in some cases class) (`taxonabund.txt`). 
    * If taxonomy was added to `framebot.fa` ahead of time, taxonomic abundance is calculated by phylum and lineage. 
    * If taxonomy was not added to `framebot.fa` ahead of time, taxonomic abundance of lineage is shown twice.

---

## Multiple Samples
To see how results for multiple samples can be combined into an OTU table for community analyses, see the section **Xander Results for Multiple Samples**.
