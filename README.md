# diff_abundance
Compare OTU abundances between treatments, using negative binomial normalization method.

#!/usr/bin/Rscript

################################################################
# Script for diferential abundance of OTUs 
# MORAIS, D.K. & BRUSTOLINI, O. 2015
#
# Version v.1 (03/04/2015)
#
# *p-values will be corrected using the Benjaming-Hochberg method.
# 
###############################################################

library("edgeR")
#Define the experiment desing
experiments <- read.delim("data.frame.txt", stringsAsFactors = FALSE) #"data.frame.txt" is a text file similar to a qiime mapping file. It has the following structure: (ignore the # and the ")
#"
#SampleID	Group	Description
#1BF	BF_cntrl	1BF
#2BF	BF_cntrl	2BF
#3BF	BF_cntrl	3BF
#4BF	BF_bcp	4BF
#5BF	BF_bcp	5BF
#6BF	BF_bcp	6BF
#7BF	BF_oil	7BF
#8BF	BF_oil	8BF
#9BF	BF_oil	9BF
#10BF	BF_comb	10BF
#11BF	BF_comb	11BF
#12BF	BF_comb	12BF
#"

grupo <- experiments[,2] # This is to select the treatments that Holmes method will compare. In this case, the 2nd column.

#Define your count matrix
data <- read.table(file="otu_table.txt",sep="\t",header=TRUE,row.names=1) # "otu_table.txt" is a comma separated file converted from an "otu_table.biom" and edited to make "R" likes it. Here is an example.
#"
#OTU_ID	12BF	6BF	9BF	11BF	7BF	10BF	1BF	3BF	8BF	2BF	4BF	5BF
#OTU_105	1.0	9.0	0.0	0.0	5.0	2.0	7.0	10.0	3.0	4.0	2.0	6.0
#OTU_144	8.0	6.0	1.0	1.0	5.0	1.0	5.0	7.0	8.0	3.0	4.0	6.0
#OTU_67	376.0	273.0	500.0	261.0	396.0	294.0	437.0	271.0	314.0	326.0	424.0	469.0
#OTU_255	34.0	43.0	12.0	14.0	24.0	18.0	34.0	42.0	22.0	30.0	36.0	31.0
#"

#Rearrange your data. As the data came from qiime, the count matrix was out of order. Rearrange it according to your experimental design.
mdat <- data[,c(7,10,8,11,12,2,5,9,3,6,4,1)] #This is a command to rearrange the otu_table.txt column order so the repetitions can be side by side.

#Start a loop to normalize and compare treatments 2 by 2 and detect differentially abundant OTUS between your treatments.
for (i in seq (1,7,3))
	for (j in seq ((i+3),10,3))
	{
			mat <- cbind (mdat[,c(i,i+1,+i+2)],mdat[,c(j,j+1,j+2)])
			des <- c(rep (grupo[i],3), rep(grupo[j],3))
			d <- DGEList (counts = mat, group = des)
			d <- calcNormFactors(d, method="RLE")
			d <- estimateCommonDisp(d)
			d <- estimateTagwiseDisp(d, prior.df = getPriorN(d), grid.length = 1000)
			de <- exactTest(d, pair=c(grupo[i],grupo[j]), dispersion="auto")
		
			dt <- topTags (de, n=20000, adjust.method="BH")
			tab1 <- dt[dt$table$FDR < 0.05,]
			tab <- dt$table
			write.csv2 (tab, paste("./tabelasBH/",grupo[i], "_vs_", grupo[j],".csv", sep=""))
	        write.csv2 (tab1, paste("./tabelasBH/","s",grupo[i], "_vs1_", grupo[j],".csv", sep=""))
	}

#References
# http://www.bioconductor.org/packages/2.9/bioc/html/edgeR.html
# McMurdie PJ, Holmes S (2014) Waste Not, Want Not: Why Rarefying Microbiome Data Is Inadmissible. PLoS Comput Biol 10(4): e1003531. doi:10.1371/journal.pcbi.1003531
