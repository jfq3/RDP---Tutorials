## Introduction
Xander, part of the [RDPTools](https://github.com/rdpstaff/RDPTools) package (Cole *et. al.*, 2014), is a novel method for targeted assembly of specific protein-coding genes from metagenomic data (Wang *et. al.*, 2015). It does this using a graph structure combining both de Bruijn graphs and protein Hidden Markoff Models (HMMs). The inclusion of HMM information guides and speeds the assembly, with concomitant gene annotation.

Xander includes post-assembly steps to quality filter the sequences and find closest matches in a reference data base using [FrameBot](https://github.com/rdpstaff/Framebot) (Wang *et. al.*, 2013). Xander comes preconfigured with HMMs and reference databases for the genes *amoA_ AOA, AmoA_ AOB, nifH, nirK, nirS, norB_ cNor, norB_ qNor, nosZ,* and *nosZ_ a2*. Users may add capability for other genes using information conveniently collected in the [FunGene repository](http://fungene.cme.msu.edu/) (Fish *et. al.*, 2013).

This document provides instructions and exercises on how to use Xander, how to add new gene resource files, and how to collect results from multiple samples into files that can be used to fully populate an experiment-level phyloseq object (McMurdie and Holmes, 2013) for downstream community analysis using R (R Core Team, 2017).

## References
**Cole, J. R., Wang, Q., Fish, J. A., Chai, B. L., McGarrell, D. M., Sun, Y. N., Brown, C. T., Porras-Alfaro, A., Kuske, C.R., and Tiedje, J. M.** (2014). Ribosomal Database Project: data and tools for high throughput rRNA analysis. Nucleic Acids Research, 42(D1), D633-D642. URL: https://academic.oup.com/nar/article/42/D1/D633/1063201 doi:10.1093/nar/gkt1244

**Fish, J. A., Chai, B., Wang, Q., Sun, Y., Brown, C. T., Tiedje, J. M., and Cole, J. R.** (2013). FunGene: the functional gene pipeline and repository. Frontiers in Microbiology, 4. URL: https://www.frontiersin.org/articles/10.3389/fmicb.2013.00291/full  doi:10.3389/fmicb.2013.00291.

**McMurdie, P. J., and Holmes, S.** (2013). Phyloseq: An R package for reproducible interactive analysis and graphics of microbiome census data. Plos One, 8(4). URL: http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0061217 doi:10.1371/journal.pone.0061217.

**R Core Team** (2017). R: A language and environment for statistical computing. R Foundation  for Statistical Computing, Vienna, Austria. URL https://www.R-project.org/.

**Wang, Q., J. A . Fish, M . Gilman, Y . Sun, C. T. B rown, J. M . Tiedje and J. R . Cole.** (2015). Xander: Employing a Novel Method for Efficient Gene-Targeted Metagenomic Assembly. Microbiome 3:32. DOI: 10.1186/ s40168-015-0093-6. URL: https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-015-0093-6 content/3/1/32.

**Wang, Q., Quensen, J. F., Fish, J. A., Lee, T. K., Sun, Y. N., Tiedje, J. M., and Cole, J. R.** (2013). Ecological patterns of nifH genes in four terrestrial climatic zones explored with targeted metagenomics using FrameBot, a new informatics tool. MBio, 4(5), e00592-00513. URL: http://mbio.asm.org/content/4/5/e00592-13.full.pdf doi:10.1128/mBio.00592-13.
