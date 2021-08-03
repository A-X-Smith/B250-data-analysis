# Periodicity (RiboWaltz)

> **_IMPORTANT:_** `$BASE_DIR` has to be specified in your `.bash_profile`. Details [here](docs/0_before_you_start.md)

> **_IMPORTANT 2:_** R Package [RiboWaltz](https://github.com/LabTranslationalArchitectomics/riboWaltz) is required. 

RiboWaltz requires bam files with **transcriptome** coordinates (not the genome!), e.g. 

```
VH00211:4:AAAKJH7M5:1:1308:66460:17414_GGTCTGATCA       256     ENST00000456328.2       227     0       32M     *       0       0       CTAGAGTGGGATGGGCCATTGTTCATCTTCTG    CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC        NH:i:12 HI:i:8
VH00211:4:AAAKJH7M5:1:2207:35917:30230_TCCTGGATCA       256     ENST00000456328.2       361     0       30M     *       0       0       TGTGTGGTGATGCCAGGCATGCCCTTCCCT  CCCCCCCCCCCCCCCCCCCCCCCCCCCCCC  NH:i:21 HI:i:17
VH00211:4:AAAKJH7M5:1:1410:39666:48214_TGGTGGATCA       256     ENST00000456328.2       660     0       30M     *       0       0       CACCCCGAGATCACATTTCTCACTGCCTTC  CCCCCCCCC;CCCCC;CCCC;CCCCCCCC;  NH:i:16 HI:i:4
VH00211:4:AAAKJH7M5:1:1308:66460:17414_GGTCTGATCA       256     ENST00000515242.2       224     0       32M     *       0       0       CTAGAGTGGGATGGGCCATTGTTCATCTTCTG
```

where `ENST00000456328.2` is the transcript_id (3rd column), and `227` is the coordinate (1-based) of the 5' end of the read within the transcript sequence (4th column).

If alignment is performed as described [here](/docs/5_align.md), these bam files will be created by STAR and written in this directory:

```
$BASE_DIR/22276/analysis/output/alignments/toTranscriptome
```

## 1. Create input dir

```
mkdir -p $BASE_SIE/22276/analysis/input/periodicity_bam/all_unique/
```
## 2. Create symlinks

```
for f in $(ls $BASE_DIR/22276/analysis/output/alignments/toTranscriptome/*_toTranscriptome_dedup.bam); 
    do fn=$(basename $f); 
    fn=${fn%_toTranscriptome_dedup.bam}.bam; 
    echo "ln -s $f $BASE_DIR/22276/analysis/input/periodicity_bam/all_unique/$fn"; 
done
```

## 3. Run RiboWaltz

```
module load gcc/7.2.0
module load R/3.6.2
Rscript $BASE_DIR/software/rp_analysis/periodicity.r 20910 mm10 all_unique
```

## 4. RiboWaltz stats

Create ribowaltz_stats file:  `vim $BASE_DIR/20910/analysis/output/periodicity/all_unique/ribowaltz_stats.tsv`

copy & paste output of `bamtolist` function.
Do the following vim commands:

```
:%s/.bam//g
:%s/Reading //g
:%s/ M/M/g
:%s/Input reads: //g
:%s/Output reads: //g
:%s/(.*/
:%s/ //g
:%s/\n/\t/g
:%s/\t\t/\t/g
:%s/\tS/\rS/g
:%s/Good.*//g
:%s/Great.*/0M/g
:%s/Done.*//g
```

Add header: `sample  input_reads not_in_annotation   negative_strand output_reads   cds_reads   inframe_reads`
