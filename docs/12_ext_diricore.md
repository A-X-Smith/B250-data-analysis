# Ext Diricore (P-site signalling)

> **_IMPORTANT:_** This step requires [plastid](https://plastid.readthedocs.io/en/latest/). To install, run the command: `source activate p3 && pip install plastid` 

## 0. Set up plastid

Install plastid: `pip install plastid`


Create reference for human (for mouse it's the same, just take the annotation for mouse - replace `hg19` with `mm10`): 

```
bsub -q long -R "rusage[mem=50G]" metagene generate $BASE_DIR/static/hg19/plastid --landmark cds_start --annotation_files $BASE_DIR/static/hg19/gencode.annotation.gtf --downstream 200
```

This will create 2 files `plastid_rois.bed` and `plastid_rois.txt` at the static directory: `$BASE_DIR/static/hg19/`



## 1. Get sequences from bam

Get sequences: 

```
bsub -R "rusage[mem=10G]" $BASE_DIR/software/ext_diricore/1_get_seq_from_bam.sh 22276 all_unique
```

## 2. Index bam files 

```
for f in $(ls $BASE_DIR/22276/analysis/output/alignments/toGenome/*_toGenome_dedup.bam); do echo "samtools index $f"; done
```

## 3. Calculate offsets by plastid 

```
for f in $(ls $BASE_DIR/22276/analysis/output/alignments/toGenome/*_dedup.bam); do fn=$(basename $f); fn=${fn%_toGenome_dedup.bam};  echo "bsub -q medium  -R "rusage[mem=10G]" psite $BASE_DIR/static/hg19/plastid_rois.txt  $BASE_DIR/22276/analysis/output/plastid/p_offsets/all_unique/$fn --min_length 25 --max_length 35 --require_upstream --count_files $f --title "$fn""; done
``` 

This will calculate the offset for each read length (per sample), and create some images + data files in this directory: `$BASE_DIR/22276/analysis/output/plastid/p_offsets/`

Copy the plots from the cluster (run it locally on the laptop): 

```
scp -r USERNAME@odcf-cn34u03s10:BASE_DIR/22276/analysis/output/figures/p_offsets ~/analysis/22276/figures/
```

`USERNAME` has to be replaced with the real username (e.g. `e984a`), and `BASE_DIR` has to be replaced with the real BASE_DIR (e.g. `/omics/groups/OE0532/internal/from_snapshot/`)

The plot will look like that:

![](pics/plastid.png) 

## 4. Manually re-format offset files

Now we need to extract the offset and eliminate non-relevant data from the plastid output.
 
Create data dir: 

```
mkdir -p $BASE_DIR/22276/analysis/output/ext_diricore/all_unique/p_offset
```

Copy data files:

```
for f in $(ls $BASE_DIR/22276/analysis/output/plastid/p_offsets/all_unique/*p_offsets.txt); do echo "cp $f $BASE_DIR/22276/analysis/output/ext_diricore/all_unique/p_offset/"; done
```

Remove lines starting with `#`:

```
for f in $(ls $BASE_DIR/22276/analysis/output/ext_diricore/all_unique/p_offset/*); do echo "cat $f | grep -v '^#' | grep -v default > $f"1; echo "mv $f"1 $f; done
```