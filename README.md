# LAVA partitioning algorithm
Partitioning algorithm used to create the blocking for [LAVA](https://github.com/josefin-werme/lava). Main program must first be compiled from source using provided makefile.

Processing is done per chromosome, and requires PLINK data files of the reference data used for LD estimation to be split by chromosome. The main program processes the reference data into a sequence of break points on the chromosome, recursively splitting the largest block defined by the current break points (starting with the whole chromosome) by selecting a new break point that minimizes local LD between the resulting two new blocks. The ldblock.r script can then be used to process the breakpoint file into blocks to be used for input in LAVA (or other tools), applying optional filtering to obtain different blocking solutions. 

There are a number of parameters that determine how break points are defined (see included manual for more details). The most central of these are the SNP window size and the MAF threshold. The SNP window size determines for a given SNP how far back and forward LD is computed with other SNPs (in number of SNPs). Higher values for this window result in more longer range LD being considered, but this comes at the expense of less sensitivity to more local differences in LD (as well as computational burden). The MAF threshold determines which SNPs to include in the primary computation, filtering out SNPs with MAF below the threshold. Note that filtered out SNPs are disregarded in further computation (except during the optional refine step, see manual), so eg. the window size parameter is applied after these SNPs are filtered out.

The algorithm will keep splitting the current set of blocks until no further valid break points can be found. This is designed to keep the size of the resulting blocks relatively even in the number of SNPs per block, to reduce the risk of large discrepancies in statistical power between blocks in subsequent analysis. The two central parameters controlling this are the minimum block size (in number of SNPs, after MAF filtering) and the maximum LD metric value. When trying to split a block, potential breakpoints (defined by two adjacent SNPs) are considered invalid if splitting the block there results in blocks smaller than the minimum size (and as such, once a block drops below twice the minimum size it will not be split any further). 

For each potential breakpoint, an LD metric is then computed as the mean squared value of all correlations (capped by the SNP window size) between SNPs on different sides of that potential breakpoint, and is considered invalid if this mean r-squared value exceeds the maximum LD metric value specified. This therefore controls the level of dependency between adjacent blocks that is considered acceptable, and prevents further splitting when resulting blocks would not be sufficiently independent. Note that the breakpoint output file registers the metric values for all breakpoints for later inspection. Moreover, the ldblock.r script allows for post-hoc filtering on the maximum metric value. As such, it is possible to generate an initial list of breakpoints at a high maximum LD metric value, and decide on desired level of maximum dependency later. 

The LD blocks generated and used for the primary LAVA paper are included here as well. These were generated at default values for the blocking algorithm except with the mininum block size set to 2500. No further post-hoc filtering was applied. 
