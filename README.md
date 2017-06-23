# Fungal ITS analysis tutorial at Microbiome Bioinformatics With QIIME 2 (June 23 2017)

This tutorial was developed at the [Microbiome Bioinformatics With QIIME 2 workshop in Las Vegas, NV in June 2017](https://workshops.qiime.org/qiime-2-workshop-2017-06-21/). This has been tested with QIIME 2 2017.6, but is not guaranteed to work outside of this workshop. If you're looking for the QIIME 2 documentation, see https://docs.qiime2.org. If you're interested in attending a QIIME 2 workshop, see https://workshops.qiime2.org.

# Fit a classifier for the [UNITE Database](https://unite.ut.ee)

```
wget https://unite.ut.ee/sh_files/sh_qiime_release_20.11.2016.zip
```

```
unzip sh_qiime_release_20.11.2016.zip
```

```
cd sh_qiime_release_20.11.2016
```

```
qiime tools import \
 --type FeatureData[Sequence] \
 --input-path sh_refs_qiime_ver7_99_20.11.2016.fasta \
 --output-path unite-ver7-99-seqs-20.11.2016.qza
```

```
qiime tools import \
 --type FeatureData[Taxonomy] \
 --input-path sh_taxonomy_qiime_ver7_99_20.11.2016.txt \
 --output-path unite-ver7-99-tax-20.11.2016.qza \
 --source-format HeaderlessTSVTaxonomyFormat
```

```
qiime feature-classifier fit-classifier-naive-bayes \
 --i-reference-reads unite-ver7-99-seqs-20.11.2016.qza \
 --i-reference-taxonomy unite-ver7-99-tax-20.11.2016.qza \
 --o-classifier unite-ver7-99-classifier-20.11.2016.qza
```

# Download an ITS mock community from [mockrobiota](http://msystems.asm.org/content/1/5/e00062-16)

```
wget -O "mock-25-sample-metadata.tsv" https://raw.githubusercontent.com/caporaso-lab/mockrobiota/master/data/mock-25/sample-metadata.tsv
wget https://s3-us-west-2.amazonaws.com/mockrobiota/latest/mock-25/mock-forward-read.fastq.gz
```

Create a file, `fastqmanifest.csv`, with the following two lines:

```
sample-id,absolute-filepath,direction
Mock.1,$PWD/mock-forward-read.fastq.gz,forward
```

```
qiime tools import \
 --type SampleData[SequencesWithQuality] \
 --input-path fastqmanifest.csv \
 --output-path demux.qza \
 --source-format SingleEndFastqManifestPhred33
```

```
qiime demux summarize \
 --i-data demux.qza \
 --o-visualization demux.qzv
```

In the following `denoise-single` command, we want to provide ``qiime dada2 denoise-single``, but [a bug was discovered](https://github.com/qiime2/q2-dada2/issues/67) that currently prevents that.

```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left 10 \
  --p-trunc-len 140 \
  --o-representative-sequences rep-seqs.qza \
  --o-table table.qza
```

```
qiime feature-classifier classify-sklearn \
  --i-classifier unite-ver7-99-classifier-20.11.2016.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza
```

```
qiime taxa tabulate \
  --i-data taxonomy.qza \
  --o-visualization taxonomy.qzv
```

```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file mock-25-sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```
