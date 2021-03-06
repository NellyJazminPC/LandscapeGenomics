## EXERCISE 4: Environmental association analysis

Once you have your SNPs from Stacks (or another program), you are ready for data analysis. Here, I will show two environmental association analyses with different purposes. The first, called redundancy analysis, is an ordination method that can be used to understand what geographic or environmental factors are associated with the overall patterns of genetic variation on the landscape. The second approach uses linear mixed modeling to identify loci that are exceptionally associated with the environment and thus potential targets of natural selection along the gradient. In each case, there are other methods that should be considered. In addition to redundancy analysis, one should also consider the R package [BEDASSLE](http://cran.r-project.org/web/packages/BEDASSLE/index.html), and in addition to linear mixed models, one should also consider [BayEnv2](http://gcbias.org/bayenv/) and perhaps *F*st outlier methods such as [BayeScan](http://cmpg.unibe.ch/software/BayeScan/).

### Redundancy analysis

#### Basic redundancy analysis
*Redundancy analysis* (RDA) is a form of constrained ordination that examines how much of the variation in one set of variables explains the variation in another set of variables. It is the multivariate analog of simple linear regression. Redundancy analysis is based on similar principles as principal components analysis and thus makes similar assumptions about the data. It is appropriate when the expected relationship between dependent and independent variables is linear (*e.g.* climate and allele frequency). If the expected relationship is Gaussian (*e.g.* climate and species abundance), then canonical correspondence analysis is more appropriate, which can be implemented as described below for redundancy analysis, but by replacing all `rda` functions with `cca`. We will use the [VEGAN package](http://cran.r-project.org/web/packages/vegan/index.html).

Open RStudio by double-clicking the icon on the Desktop. In the top left, there is a space to create a script. Let's use that instead of the command line in the bottom left. You can Run each line of the script as you type or enter this whole section and then Run.

1. To begin the building the script type the following to set the working directoy

		setwd("/home/genomics/Desktop/RDA/")

2. Load the vegan package and data files

		library(vegan)
		GenData = read.table(file="rdaSNPs.txt", header = T, row.names=1)
		ClimData = read.table(file="ClimData.txt", header = T, row.names=1)

3. Now the genetic data are stored as `GenData` and the climate data and spatial coordinates as `ClimData`. The data files must be organized so that each row represents an individual and that the individuals are in the same order in both the file with genetic data and that with climate data. `ClimData` contains the geographic coordinates (*i.e.* latitude and longitude) of the samples and five climate variables from the USGS database (http://forest.moscowfsl.wsu.edu/climate/customData/) for the site at which the sample was found. `GenData` contains SNP frequencies for each individual sample, but it needs to be transposed so that individuals are in rows. 

		tGenData <- t(GenData)

4. Run a basic redundancy analysis by specifying the equation with the *unconstrained matrix* (dependent variables, *e.g.* genetic data), the *constrained matrix* (independent variables, *e.g.* climate data and geographic coordinates), and the file where the constraining variables are found. You can specify all or some of the climate variables, but let's try just including climate variables without geographic ones.

		RDA <- rda(tGenData ~ tmax + tmin + gsp + smrsprpb + dd5, ClimData)

	This command says, run `rda` with all `GenData` as dependent variables and the 8 climate variables as independent variables, which are found in `ClimData`, and put the results in a temporary place called RDA.

4. Then we can test if the relationship between climate and genotype is significant.

		anova(RDA)
	
	The output looks a lot like an ANOVA table and the function is even called `anova`, but in fact it is a *permutation test*, in which the rows of the unconstrained matrix are randomized repeatedly across some number of permutations. If the observed relationship is stronger than the randomly permuted relationships (*e.g.* at alpha = 0.05), then the relationship is significant. Focus on the *P*-value and ignore *F* and the other components, which are not the same as what you may be familiar with in ANOVA. What is the *P*-value? Can you reject the null hypothesis that climate does not explain genetic variation?

5. To know which climate variables are most important in shaping genetic variation, we can make a *biplot*.
	
		plot(RDA)

	The black (open) points are each individual (genotypes) displayed in RDA space and the blue vectors show how climate variables fall along that RDA space. The longest vectors along each RDA axis are most important in explaining variation in genotypes along that axis. Which climate variables are most important in these data?

#### Partial redundancy analysis

In the previous section, we found a significant relationship among climate and genetic variation, and we identified climate variables that might be most important in explaining that relationship. However, climate and genotype are confounded by geographic location. Nearby locations have similar climate and nearby locations might have similar genotypes if mating is more likely locally/regionally. We can statistically control for this fact by doing a *partial redundancy analysis*, where geography (latitude and longitude) are specified as a third matrix, the *conditioned matrix*. The analysis then "controls for," "partials out," or "conditions on" geographic location. Geography could be defined simply as latitude and longitude or could be made more complex with polynomials (I use up to third order polynomials, normally) or other approaches such as Moran's Eigenvector Maps (MEM or PCNM) can be used. Here, we will keep it simple.

1. Run a partial redundancy analysis of genotype on climate, conditioned on geography.

		pRDA1<-rda(tGenData ~ tmax + tmin + gsp + smrsprpb + dd5 + Condition(Lat + Lon), ClimData)

2. Now, run the permutation test and make a biplot.

		anova(pRDA1)
		plot(pRDA1)

How do the results of this analysis differ from the previous section What does that suggest?


#### Partitioning variance components
Now that we know climate significantly explains genetic variation, even after controlling for spatial coordinates, we might want to know how much of the genetic variation is uniquely explained by climate, how much is uniquely explained by geography, and how much is explained by some joint effect of the two. 

1. To partition variance, we need to run three models: a full model with all climate and geographic variables as explanatory variables, a partial model in which geography explains genetic data conditioned on climate, and a partial model where climate explains genetic data conditioned on geography. We already did the last one, so let's do the first two.

		pRDA2<-rda(tGenData ~ Lat + Lon + Condition(tmax + tmin + gsp + smrsprpb + dd5), ClimData)
		RDAfull<-rda(tGenData ~ Lat + Lon + tmax + tmin + gsp + smrsprpb + dd5, ClimData)

2. First, get the variance components from the full model by typing

		RDAfull

	In the section called "Partitioning of variance," you will see total, constrained, and unconstrained values of inertia and their proportions of total inertia. *Inertia* is essentially variance. The proportion of total variance explained by the constrained matrix (climate and geography) appears very small. However, even if the climate and geography perfectly explain genetic variation in reality, the proportion of total variance explained by the constrained matrix can be substantially less than one. Therefore, some statisticians recommend focusing on partitioning the total *explainable* variance (and not overinterpreting variance partitioning in general). The total explainable variance is the inertia for the constrained matrix of the full model. Add this number to the VariancePartitioning.ods file in the RDA folder (double-click to open).

3. Now, we need to find how much of the explainable variance is explained by climate versus geography versus their joint effect. First, retrieve the variance partitioning output from the partial model conditioned on geography (recall that we ran this model earlier and put the results in `pRDA1`)

		pRDA1

	The variance explained purely by climate is the inertia value for the constrained matrix (climate), which represents that variance explained by climate after removing the variance explained by the conditioned matrix (geography). Repeat the process for the partial model conditioned on climate (`pRDA2`) and report the variance explained purely by climate and purely by geography in the VariancePartitioning.ods table.

		pRDA2

4. To calculate the joint effect of climate and geography, or in other words, the proportion of variance in which climate and geography cannot be separated due to collinearity, simply subtract the pure effects from the total explainable variance.

5. Finally, express the variances as proportions of the total explainable variance in the table.


#### Useful links
Oksanen J. [vegan: Community Ecology Package.](http://cran.r-project.org/web/packages/vegan/index.html) 

Oksanen J (2012) [Constrained ordination: tutorial with R and vegan.](http://cc.oulu.fi/~jarioksa/opetus/metodi/sessio2.pdf)

Oksanen J (2012) [vegan: an Introduction to Ordination.](http://cran.r-project.org/web/packages/vegan/vignettes/intro-vegan.pdf)

Oksanen J, Blanchet FG, Kindt R, Legendre P, Minchin PR, O'Hara RB, Simpson GL, Solymos P, Stevens MHH, Wagner, H (2012) [Package 'vegan.'](http://cran.r-project.org/web/packages/vegan/vegan.pdf) 

Palmer M. [Ordination Methods for Ecologists.](http://ordination.okstate.edu/)

Palmer M. [Variance explained and variance partitioning in CCA.](http://ordination.okstate.edu/varpar.html)

####Useful references
Borcard D, Legendre P, Drapeau P (1992) Partialling out the spatial component of ecological variation. *Ecology* 73, 1045-1055.

Bradburd GS, Ralph PL, Coop GM (2013) Disentangling the effects of geographic and ecological isolation on genetic differentiation. *Evolution* 67, 3258-3273.

Gugger PF, Ikegami M, Sork VL (2013) Influence of late Quaternary climate change on present patterns of genetic variation in valley oak, *Quercus lobata* Nee. *Molecular Ecology* 22, 3598-3612.

Legendre P, Legendre L (1998) *Numerical ecology*, 2nd English edn. Elsevier, Amsterdam.

Liu Q (1997) Variation partitioning by partial redundancy analysis (RDA). *Environmetrics* 8, 75-85.

Okland RH (1999) On the variation explained by ordination and constrained ordination axes. *Journal of Vegetation Science* 10, 131-136.

Peres-Neto PR, Legendre P (2010) Estimating and controlling for spatial structure in the study of ecological communities. *Global Ecology and Biogeography* 19, 174-184.

Smouse PE, Williams RC (1982) Multivariate analysis of HLA-disease associations. *Biometrics* 38, 757-768.

Westfall RD, Conkle MT (1992) Allozyme markers in breeding zone designation. *Population Genetics of Forest Trees* 42, 279-309.


### Linear mixed model (EMMAX)
Identifying genetic variation involved in adaptation to local environments in natural populations is an important topic in conservation genetics. Population genetic variation is influenced by demography, mating patterns, and natural selection, each of which is shaped by the environment. Theoretically, demographic changes and gene flow ("neutral" processes) affect genetic variation throughout the genome, whereas natural selection affects a small number of genes. To disentangle which parts of the genome are under natural selection by the environment, we can use genome-wide data sets to identify genes that are extremely associated with environmental variables after accounting for the background association of genetic variation with the environment due to demographic and mating patterns. There are several approaches to *environmental association analysis* (*e.g.* BayEnv2), but here we borrow an approach developed in another context. 

[EMMAX](http://genome.sph.umich.edu/wiki/EMMAX) is an implementation of a *linear mixed model* originally developed to associate genetic variation (*e.g.* SNPs) with phenotypic variation after accounting for kinship among individuals (**y** = *X*B + *Z*u + **e**). EMMAX empirically estimates pairwise kinship using all the input genetic variation, which is then factored out in testing the association of individual genetic variants with phenotypic variation. The resulting significant associations are candidate variants that may play a role in the genetic basis of the phenotype, when the phenotypic variation is measured under a common environment. EMMAX is one of many approaches to perform *genome-wide association analysis* (GWAS). Others include GEMMA and TASSEL. EMMAX makes simplifying assumptions to efficiently calculate the variance components, which allows for rapid testing with large sample sizes. 

Here, we will instead use this approach as a means of testing for SNPs under selection by climate in natural populations. If SNP frequency is strongly associated with a climate variable (*e.g.* minimum temperature) measured where the organism was collected, after accounting for the fact that related individuals occur near each other, then that SNP frequency could be governed by selective forces related to that climate variable. Genetic variation overall can be associated with climate due to phylogeographic history or local mating patterns clustering genetically similar individuals in climatically similar areas, and thus it is critical to account for the degree of relatedness among individuals (*i.e.* kinship) when performing this test. EMMAX conveniently applies a robust and rapidly executed implementation of a mixed model that we can use for this approach. 

1. Input files: The input file formats are TPED, TFAM and "phenotype" files, which can be created manually or in [PLINK](http://pngu.mgh.harvard.edu/~purcell/plink/data.shtml) based on the PED and MAP files output from Stacks. The TFAM columns are family ID, individual ID, paternal ID, maternal ID, sex, and phenotype (climate variable, here). The phenotype file is similar but only has family ID, individual ID, and phenotype. In our case, each family has its own ID, 1 to n. Paternal ID, maternal ID, and sex can be filled in with "NA." The TPED columns are chromosome (1-22), SNP ID, genetic distance (morgans), and SNP position (bp), followed by two columns per individual giving each allele, coded as 1, 2, or 0 for missing data. Only the SNP ID and allele columns matter here. Explore each file's structure by typing
	
		less -S allSNPs.tped
		less -S allSNPs.tfam
		less -S [phenotype].txt

2. Create the kinship matrix using all the SNP data.

		emmax-kin -v allSNPs

	allSNPs is the prefix of the TPED file I created for you. This will generate `allSNPs.aBN.kinf`, which is a matrix of pairwise kinship according the Balding-Nichols method of estimating kinship. Alternatively, you could estimate kinship by calculating the proportion of loci identical by state by adding `-s`.  

		emmax-kin -v -s allSNPs

	Both are approximate methods of determining the proportion of genetic identity by descent based on identity by state considering all loci. Look at a few lines of each kinship file

		less -S allSNPs.aBN.kinf
		less -S allSNPs.aIBS.kinf

	You can see that most individuals are unrelated (values close to 0), but some have some coancestry.
	
3. Run EMMAX to test for individual SNPs associated with climate after accounting for kinship

		emmax -v -t testSNPs -p gsp.txt -k allSNPs.aBN.kinf -o results

	I ran growing season precipitation (gsp) but you can choose another climate variable if you like. `testSNPs.tped` only contains SNPs whose minor allele frequency is greater than 10 %. There are two reasons: 1) if a very rare allele in say one individual drives the correlation with climate, then it is uninteresting, and 2) by focusing only on tests of potential interest we gain statistical power after accounting for multiple testing.
	
4. Output files: The three output files include a log of what was printed to screen (.log), a file summarizing variance components and including a "pseudoheritability" estimate in the last line (.reml), and a file with the list of tested SNPs, the beta values, the standard error of beta, and their *P*-values (.ps). The latter file indicates which individuals SNPs are significantly associated with climate after accounting for kinship. We will largely ignore the other output file, but it is worth mentioning that the "pseudoheritablity" is the proportion of phenotypic, or here climate, variance explained by the relatedness among samples (*i.e.* the kinship matrix) (usually high in this analysis with climate).
	
5. Multiple testing: Because we did so many statistical tests, we need to correct for multiple testing before drawing any conclusion from the *P*-values. One common way is using the false discovery rate (FDR) method, which is simply executed in Q-VALUE in R/Bioconductor. 

		cut -f 4 GSP.ps | Rscript -e 'library(qvalue); p<-scan("stdin"); qobj<-qvalue(p); write.qvalue(qobj,file="Qvals.txt")'

	A list of *Q*-values are given alongside the *P*-values in the same order they were input. Let's consider a *Q* < 0.05 significant. See how many are significant by sorting the file by *Q*-value.

		sort -nk2 Qvals.txt | less 
	
Now that we have a list of candidate SNPs under selection by climate, we can further investigate their genomic context, the function of the gene in which they might be found, and whether or not the SNPs lead to amino acid sequence changes, etc.


#### Useful references
Coop G, Witonsky D, Di Rienzo A, Pritchard JK (2010) Using environmental correlations to identify 
loci underlying local adaptation. *Genetics* 185, 1411-1423.

Gunther T, Goop G (2013) Robust identification of local adaptation from allele frequencies. *Genetics* 195, 205-220.

Kang HM, Sul JH, Service SK, Zaitlen NA, Kong S-Y, Freimer NM, Sabatti C, Eskin E (2010) Variance 
component model to account for sample structure in genome-wide association studies. *Nature Genetics* 42, 348-354.

Kang HM, Zaitlen NA, Wade CM, Kirby A, Heckerman D, Daly MJ, Eskin E (2008) Efficient control 
of population structure in model organism association mapping. *Genetics* 178, 1709-1723.

Storey JD, Tibshirani R (2003) Statistical significance for genomewide studies. *Proceedings of the 
National Academy of Sciences of the United States of America* 100, 9440-9445.

Yoder JB, Stanton-Geddes J, Zhou P, et al. (2014) Genomic signature of adaptation to climate in *Medicago truncatula*. *Genetics* 196, 1263-1275.

Zhou X, Stephens M (2012) Genome-wide efficient mixed-model analysis for association studies. 
*Nature Genetics* 44, 821-824.
