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
  -A "Y":"Yes" "M":"I'm" "N":"fries" \

done
```


## Now the example data

###Example data 1: Nc3, 

15-20x coverage using illumina, 1.2 Gb genome.


> outlier_analysis/ <br>
> ├── analysis <br>
> │   ├── bayescan <br>
> │   ├── baypass <br>
> │   ├── pcadapt <br>
> │   ├── summary <br>
> │   └── vcftools_fst <br>
> ├── data  <br>
> ├── programs  <br>
> └── workshop_material <br>

