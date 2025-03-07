# Rv3_SVecoevolutionary
Example samplot datasets

## some exaple code for running samplot and then sam critic

First [Samplot](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02380-5)

A metadata file ``samplot_batch_$BATCH.txt`` was preprepared with the following info per line;
CHROM START END LENGTH TYPE = information specific to the SV
HOMR1 HOMR2 HET1 HET2 HOMA1 HOMA2 = example genotypes (2x homoxygous REF, heterozygous, and homozygous ALT).

Running samplot and plotcritic in batches is recommended as this allows you to chunk your work - and you can't save your progress half way through a score run. I like to break things into groups of similar type and size, as this helps make the scoring process faster and more consistent.

If you do not have enough of one genotype, but you will want to score, then I would recommend creating a a dummy individual.

For example code for metadata file creation, and incorporation of a dummy indiividual in your plots, see [here](https://github.com/katarinastuart/Nc3_HihiSV/blob/main/Code_PDFs/2_Curation.pdf).


```
DIR=/project/rawdata/mapped_reads

for BATCH in DUP INVsmall INVbig DELsmall DELbig;

do

mkdir $(echo $BATCH)

while IFS= read -r line; do
    # Split the line into individual values
    read -r ID CHROM START END LENGTH TYPE HOMR1 HOMR2 HET1 HET2 HOMA1 HOMA2 <<< "$line"
    # Use the variables in your loop

VARIANTS=$(echo "-o" $BATCH/$ID "-c" $CHROM "-s" $START "-e" $END "-t" $TYPE "-d 100" -n $HOMR1 $HOMR2 $HET1 $HET2 $HOMA1 $HOMA2)
BAMS=$(echo "-b" ${DIR}/$HOMR1".sorted.dup.bam" ${DIR}/$HOMR2".sorted.dup.bam" ${DIR}/$HET1".sorted.dup.bam" ${DIR}/$HET2".sorted.dup.bam" ${DIR}/$HOMA1".sorted.dup.bam" ${DIR}/$HOMA2".sorted.dup.bam") 

SAMPLOT=$(echo "samplot plot" $VARIANTS $BAMS) 
echo $ID
$SAMPLOT

done < samplot_batch_$BATCH.txt

done

```

Now [PlotCritic/SV-plaudit](https://academic.oup.com/gigascience/article/7/7/giy064/5026174?login=false) wraps up the .png files into a neat package that you can open on a browser and score!


```
for BATCH in DUP INVsmall INVbig DELsmall DELbig;

do

plotcritic \
  -p ${BATCH}_plot \
  -i $BATCH/ \
  -q "Is this a structural variant?" \
  -A "Y":"Yes" "M":"Maybe you want to make a middle catagory: uncertain?" "N":"No" \

done
```

finally, you can import the variants into R to find the variant IDs that you want to keep.

```
library(dplyr)
library(stringr)
library(jsonlite)

DELasmall <- fromJSON("DELasmall_plot_report.json", flatten=TRUE)
DELabig <- fromJSON("DELabig_plot_report.json", flatten=TRUE)
DUP <- fromJSON("DUP_plot_report.json", flatten=TRUE)
INV <- fromJSON("INV_plot_report.json", flatten=TRUE)
DELb <- fromJSON("DELb_plot_report.json", flatten=TRUE)

outcomes <- rbind(DELasmall,DELabig, DUP, INV, DELb)

result <- outcomes %>% mutate(SNP_ID= str_replace_all(Image, "_", ":")) 
result <- result %>% mutate(keep = if_else(result$score =="Y", "keep", "discard")) 
keep <- result %>% filter(keep == "keep")

write.table(keep$SNP_ID, file = "curated_SVs_keep.txt", sep = "\t", row.names = FALSE, quote = FALSE, col.names=FALSE)

```

<p></p>

> :heavy_exclamation_mark: When you score your plots, you may notice that while the plots support the called SV being real, some individuals might have been assigned the wrong genotype compared to the genotype you believe their raw mapped data supports. It is always a good idea to re-genotype your SVs, and if you see a lot of miss-assigned genotypes then this means it is extra important for your dataset.

<p></p>

> :beginner: **False positives, or false negatives?**
>
> *excerpt from main manuscript*
> 
> Even under ideal scenarios, with high coverage and very good quality raw sequencing data, there will still be a fair amount of noise in your data, creating uncertainty regarding whether the SVs you have identified are true variants. It might also be important to acknowledge that some SVs that we remove from our data (because they do not meet our filtering or curation criteria) might be reflecting real underlying genetic differences between individuals that are too complex for our variant identification methods to capture.
>
> Given this, it is important to think about what false positives versus false negatives mean for your biological questions and curate and filter accordingly. Even with manual curation, probably the strictest approach for filtering population level SV data, there will always be variants you will be uncertain of. If you want to have a high confidence dataset, then treat the variants by default false and retain only if they meet strict criteria. On the other hand, if you want to keep as many true positives as possible and false negatives don’t interfere with your analysis (maybe you will be doing more downstream analysis that will help validate variants of interest) start from a default position of retaining variants, and only remove if support for the variant is very low (e.g. coverage, quality, curation plots). If you have a complimentary SNP dataset alongside your SV dataset, consider that you can use this to assist with your analysis. For example, if you need to calibrate a program based on population parameters, calculate these parameters on SNP dataset. Or if you end up focused on a small subset of SVs that are flagged as interesting, you can try validate your findings using, for example through qPCR, QTL mapping or region selective sweeps based on SNP data to validate the role of a SV in e.g. GWAS or GEA. Though note that this approach is not perfect as some disconnect between SNPs and SVs are expected.



## Now the example data

### Example data 1: Nc3 [structural variants in hihi](https://onlinelibrary.wiley.com/doi/abs/10.1111/mec.17631).

15-20x coverage using illumina, 1.2 Gb genome, avian. Aiming for <u>high confidence</u> (minimal false positives, false negatives less important) - we want to calculate load from these variants and find associations with lifetime fitness.

> ├── DUP:     <br>
> ├── INV:   <br>
> ├── DELasmall: smaller deletions with 2x examples for each genotype  <br>
> ├── DELabig:  smaller deletions with one example genotype missing (dummy indivividual data is plotted instead) <br>
> └── DELb:  <br>

### Example data 2: At3 structural variants in myna (unpublished).

15-20x coverage using illumina, 1.1 Gb genome, avian. Aiming for <u>high confidence</u> (minimal false positives, false negatives less important) - we want to calculate load from these variants and find associations with lifetime fitness.


> ├── DUP: Duplications    <br>
> ├── INV: Inversions  <br>
> ├── DELasmall: smaller deletions with 2x examples for each genotype  <br>
> ├── DELabig: larger deletions with 2x examples for each genotype <br>
> └── DELb: smaller deletions with one example genotype missing (dummy individual used)  <br>


> :heavy_exclamation_mark: I am looking to expand this list of examples as I work on more species and other datasets. I hope to add some long read curation, as well as plant example data, in the future.
