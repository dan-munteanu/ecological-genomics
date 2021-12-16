Each session, activate Qiime 2 in my home/microbiome directory

```
cd ~/dansresults
conda activate qiime2-2021.8
```

..and redirect the temporary directory

```
export TMPDIR="/data/project_data/16S/tmptmpdir"
echo $TMPDIR 
```



Import the data

```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path /data/project_data/16S/pyc_manifest \
  --input-format PairedEndFastqManifestPhred33V2 \
  --output-path demux-paired-end_full.qza
```



Generate summary plots on data quality 

```
qiime demux summarize \
  --i-data demux-paired-end.qza \         
  --o-visualization demux-pyc.qzv  
```



Denoise data with DADA2 (run in **screen** because this step is very intensive). We picked these trim/truncation locations based on quality visualization, above).

```
screen
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux-paired-end.qza \
  --p-n-threads 4 \
  --p-trim-left-f 16 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 289 \
  --p-trunc-len-r 257 \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza
```



Generate visualizations of features, feature sequences, and denoising stats

```
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file pyc_manifest

qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv

qiime metadata tabulate \
  --m-input-file denoising-stats.qza \
  --o-visualization denoising-stats.qzv
```



Microbial diversity data analysis (beyond the scope of what I wrote about, but I got the code down :~) ) 



Use a feature classifier to identify taxonomic groups (already done, but I got the code down too)



Train the classifier on the full (not subset) dataset that I used

```
qiime feature-classifier classify-sklearn \
  --i-classifier /data/project_data/16S/training-feature-classifiers/classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification full-taxonomy.qza

qiime metadata tabulate \
  --m-input-file full-taxonomy.qza \
  --o-visualization full-taxonomy.qzv 
```



Generate interactive bar plots for visualization

```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy full-taxonomy.qza \
  --m-metadata-file pyc_manifest \
  --o-visualization full-taxa-bar-plots.qzv
```



Test for differences in abundance using ANCOM (run in **screen**)

```
qiime composition add-pseudocount \
  --i-table table.qza \
  --o-composition-table comp-table.qza
 
qiime composition ancom \
  --i-table comp-table.qza \
  --m-metadata-file pyc_manifest \
  --m-metadata-column site-animal-health \
  --o-visualization full-ancom-site-animal-health.qzv 
```



Gneiss differential abundance testing

```
qiime gneiss correlation-clustering \
  --i-table table.qza \
  --o-clustering gneiss_corr_clust_hierarchy.qza
```



Gneiss differential abundance clustering heatmap

```
qiime gneiss dendrogram-heatmap \
  --i-table table.qza \
  --i-tree gneiss_corr_clust_hierarchy.qza \
  --m-metadata-file pyc_subset_manifest \
  --m-metadata-column site-animal-health \
  --p-color-map seismic \
  --o-visualization heatmap.qzv
```



File transfer .qzv 's and view in Qiime2 viewer. 