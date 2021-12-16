**Network analysis**

Collapse complete feature table to family level and export as a .biom

```
conda activate qiime2-2021.8
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 5 \
  --o-collapsed-table table-lev5.qza
  
qiime tools export \
  --input-path table-lev5.qza \
  --output-path feature-table-lev5.biom
```

Run SCNIC 'within' pipeline

```
SCNIC_analysis.py within -i feature-table-lev5.biom -o within_output/ -m sparcc
```

Run SCNIC 'modules' pipeline

```
SCNIC_analysis.py modules -i within_output-lev5/correls.txt -o modules_output-lev5/ --min_r .50 --table feature-table-lev5.biom
```

Filter the collapsed feature table based on site/animal health status and export

```
conda activate qiime2-2021.8
qiime feature-table filter-samples \
  --i-table table-lev5.qza \
  --m-metadata-file /data/project_data/16S/pyc_manifest \
  --p-where "[site-animal-health]='HH'" \
  --o-filtered-table HH-filtered-table.qza
  
qiime tools export \
  --input-path HH-filtered-table.qza \
  --output-path HH-filtered-table.biom
```

```
qiime feature-table filter-samples \
  --i-table table-lev5.qza \
  --m-metadata-file /data/project_data/16S/pyc_manifest \
  --p-where "[site-animal-health]='SH'" \
  --o-filtered-table SH-filtered-table.qza
  
qiime tools export \
  --input-path SH-filtered-table.qza \
  --output-path SH-filtered-table.biom
```

```
qiime feature-table filter-samples \
  --i-table table-lev5.qza \
  --m-metadata-file /data/project_data/16S/pyc_manifest \
  --p-where "[site-animal-health]='SS'" \
  --o-filtered-table SS-filtered-table.qza
  
qiime tools export \
  --input-path SS-filtered-table.qza \
  --output-path SS-filtered-table.biom
```

Run SCNIC 'within' pipeline for each site/animal health status group

```
SCNIC_analysis.py within -i HH-filtered-table.biom/feature-table.biom -o within_output-lev5-HH-Filtered/ -m sparcc
```

```
SCNIC_analysis.py within -i SH-filtered-table.biom/feature-table.biom -o within_output-lev5-SH-Filtered/ -m sparcc
```

```
SCNIC_analysis.py within -i SS-filtered-table.biom/feature-table.biom -o within_output-lev5-SS-Filtered/ -m sparcc
```

Export correls.txt edge list file, clean data labels for taxa, trim to include only correlations with |R|>0.50, visualize and extract network features in Gephi using just the user interface. 



**Functional profiling (Picrust2)**

Export feature table as a .biom

```
conda activate qiime2-2021.8
qiime tools export \
  table.qza \
  --output-dir feature-table.biom
```

Download sequence table from rep-seqs.qzv as a FASTA file and rename .fasta extension as .fna

Run PICRUSt2 pipeline

```
conda activate picrust2
picrust2_pipeline.py -s 1117sswd.fna -i feature-table.biom -o picrust2_out_pipeline -p 1
```

Download output files (.tsv) for KEGG orthology, EC numbers, and inferred pathways and visualize using Morpheus web tool (https://software.broadinstitute.org/morpheus/)

