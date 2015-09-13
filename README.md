

# wiggleplotr
Wiggleplotr is an R package to plot alternative transcript structures of a gene and RNA-Seq read coverage across them. A key feature of wiggleplotr is that it allows to scale introns to a fixed length. This feature greatly improves visualisation of alternative transcription events such as alternative promoter usage, alternative splicing and alternative 3’ end usage, because alternative exons are shown immediately adjacent to each other.

## Installation
Wiggleplotr R package can we installed from GitHub. It requires the following R packages to be installed: girdExtra, ggplot2.

## Usage
Wiggleplotr has two main functions: **plotTranscripts** that can be used to plot alternative transcripts of a gene, and **wiggleplotr** to plot RNA-Seq read coverage together with alternative transcripts.

### Plotting alternative transcripts
The **plotTranscripts** function has four parameters:
* `gene_exons` - list of GRanges objects, each object containing exons for one transcript. The list must have names that correspond to transcript IDs.
* `gene_cdss` (optional) - list of GRanges objects, each object containing the coding regions (CDS) of a single transcript. The list must have names that correspond to transcript IDs. If specified then coding and non-coding regions will be plotted in a different colour.
* `plotting_annotations` - data frame with the following four columns: `transcript_id`, `gene_id`, `gene_name`, and `strand`. All of the transcripts in the gene_exons and gene_cdss lists must have a matching transcript ID in the transcript_id column. This data frame is used to add gene name and strand to the transcript structure plot.
* `rescale_introns` - Specifies if the introns should be scaled to fixed length or not. (default: TRUE)
* `new_intron_length` - length (bp) of introns after scaling. (default: 50)

The `gene_exons` and `gene_cdss` lists as well as `plotting_annotations` data frame can all be constructed manually. However, an easier option is to download all of the necessary data from Ensembl biomart. Detailed instructions together with code are given here. 

#### Example: plot protein coding transcripts of the TLR4 gene
First, you need to download Ensembl transcript annotations and metadata by following this [tutorial](https://github.com/kauralasoo/wiggleplotr/blob/master/download_annotations.md). Once the annotations have been downloaded, we can load them into R.


```r
library("devtools")
library("GenomicFeatures")
library("dplyr")
load_all("../wiggleplotr/")
```

```
## Loading wiggleplotr
```

```
## Warning: character(0)
```

```
## Warning: character(0)
```

```
## Warning: character(0)
```

```
## Warning: character(0)
```

```
## Warning: character(0)
```

```
## Warning: character(0)
```

```r
transcript_metadata = readRDS("transcript_metadata.rds")
txdb79 = loadDb("data/TranscriptDb_GRCh38_79.db")
```
Next, we extract exon and coding sequence (CDS) coordinates from the transcript database

```r
exons = exonsBy(txdb79, by = "tx", use.names = TRUE)
cdss = cdsBy(txdb79, by = "tx", use.names = TRUE)
```
We can also find all protein coding transcript of the TLR4 gene

```r
tlr4_tx = dplyr::filter(transcript_metadata, gene_name == "TLR4", transcript_biotype == "protein_coding")
tlr4_tx
```

```
##     transcript_id         gene_id gene_name strand   gene_biotype
## 1 ENST00000394487 ENSG00000136869      TLR4      1 protein_coding
## 2 ENST00000355622 ENSG00000136869      TLR4      1 protein_coding
##   transcript_biotype
## 1     protein_coding
## 2     protein_coding
```
Finally, we plot the exon structure of these two transcripts:

```r
tlr4_txids = tlr4_tx$transcript_id
plotTranscripts(exons[tlr4_txids], cdss[tlr4_txids], transcript_metadata, rescale_introns = FALSE)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

### Plotting RNA-Seq read coverage
The **wiggleplotr** command works similarly to plotTranscripts, but it requires an additional data frame that describes where the bigWig files containing the read coverage are located and how to the data should be plotted. If you do not have bigWig files yet, then you can learn how to create them here.

The **wiggleplotr** function has eight parameters:
* `exons` - list of GRanges objects, each object containing exons for one transcript. The list must have names that correspond to transcript IDs.
* `cdss` (optional) - list of GRanges objects, each object containing the coding regions (CDS) of a single transcript. The list must have names that correspond to transcript IDs. If specified then coding and non-coding regions will be plotted in a different colour.
* `sample_data` - data frame with the metadata for the bigWig read coverage files. Must contain the following columns: `sample_id` (unique id for each sample, if multiple bigWig files have the same sample id they will be overlayed on the same plot, will be used as facet label on the right), `bigWig` (path to the bigWig file), `library_size` (normalisation factor for each sample, useful if different samples sequences to different depth and bigWig files not normalised for that), `tissue` (additional column to group samples into, is used as the colour of the coverage track)
* `transcript_annotations` - data frame with the following four columns: `transcript_id`, `gene_id`, `gene_name`, and `strand`. All of the transcripts in the exons and cdss lists must have a matching transcript ID in the transcript_id column. This data frame is used to add gene name and strand to the transcript structure plot.
* `rescale_introns` - Specifies if the introns should be scaled to fixed length or not. (default: TRUE)
* `new_intron_length` - length (bp) of introns after scaling. (default: 50)
* `plot_fraction` - Size of the random sub-sample of points used to plot coverage (between 0 and 1). Smaller values make plotting significantly faster. (default: 0.1)
* `heights` - Specifies the proportion of the vertical height that is dedicated to coverage plots (first value) relative to the transcript annotations (second value). (default: c(0.75,0.25))




