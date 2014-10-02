## fulltext ~ Get full text across all da journals

[![Build Status](https://api.travis-ci.org/ropensci/fulltext.png)](https://travis-ci.org/ropensci/fulltext)

rOpenSci has a number of R packages to get either full text, metadata, or both from various publishers. The goal of `fulltext` is to integrate these packages to create a single interface to many data sources.

Data sources in `fulltext`:

* Public Library of Science (PLOS) - via the `rplos` package
* Biomed Central - via the `bmc` package
* eLife - via the `elife` package
* We __will__ add more, as publishers open up, and as we have time...See the [master list here](https://github.com/ropensci/fulltext/issues/4#issuecomment-52376743)

We'd love your feedback. Let us know what you think at info@ropensci.org, or in [the issue tracker](https://github.com/ropensci/fulltext/issues).

## Installation

You need Rcampdf, which is not on CRAN, but you can install easily via


```r
install.packages("Rcampdf", repos = "http://datacube.wu.ac.at/", type = "source")
```

Then instal `fulltext`


```r
devtools::install_github(c("ropensci/rplos", "ropensci/bmc", "ropensci/elife"))
devtools::install_github("ropensci/fulltext")
```


```r
library('fulltext')
```

## fulltext interface

All functions are prefixed with `ft_`.

### Search

`ft_search()` - get metadata on a search query.


```r
ft_search(query='ecology', from='plos')
```

```
## Query:
##   [ecology] 
## Found:
##   [PLoS: 23356; BMC: 0; Crossref: 0; Entrez: 0] 
## Returned:
##   [PLoS: 10; BMC: 0; Crossref: 0; Entrez: 0]
```

### Text

`ft_get()` - get full or partial text of articles.


```r
ft_get(ids='10.1371/journal.pone.0086169', from='plos')
```

```
## [1] full-text articles retrieved 
## Min. Length: 111132 - Max. Length: 111132 
## IDs: 10.1371/journal.pone.0086169 ... 
## 
## NOTE: extract xml strings like output$source['<doi>']
```

### Cache

__in development__

When dealing with full text data, you can get a lot quickly, and it can take a long time to get. That's where caching comes in. And after you pull down a bunch of data, if you do so within the R session, you don't want to lose that data if the session crashes, etc. When you search you _will be able to_ (i.e., not ready yet) optionally cache the raw JSON/XML/etc. of each request locally - when you do that exact search again we'll just give you the local data - unless of course you want new data, which you can do.


```r
ft_search(query='reproducible science', from=c('plos','elife'), cache=TRUE)
```

### Visualize

__in development__

`ft_plot()` is a convenience function to plot the results of a call to `ft_search()`. There are a variety of options available, including common ways that one vizualizes full text data.


```r
res <- ft_search(query='reproducible science', from=c('plos','elife'))
ft_plot(res, method='somemethod')
```

(cool plot)

### pdf to text

There are going to be cases in which some results you find in `ft_search()` have full text available in text, xml, or other machine readable formats, but some may be open access, but only in pdf format. We have a series of convenience functions in this package to help extract text from pdfs, both locally and remotely.

Locally, using code adapted from the package `tm`, and various pdf to text parsing backends


```r
pdf1 <- system.file("examples", "example1.pdf", package = "fulltext")

(res_rcamp <- ft_extract(pdf1, "rcamp"))
```

```
## <document>/Users/sacmac/Library/R/3.1/library/fulltext/examples/example1.pdf
##   File size: 909801 bytes
##   Pages: 909801 bytes
##   Producer: Acrobat Distiller 10.1.5 (Windows)
##   Creation date: 909801 bytes
```

```r
(res_gs <- ft_extract(pdf1, "gs"))
```

```
## <document>/Users/sacmac/Library/R/3.1/library/fulltext/examples/example1.pdf
##   Title: ecsp-04-08-07 1..16
##   Producer: Acrobat Distiller 10.1.5 (Windows)
##   Creation date: 2013-08-16
```

```r
(res_xpdf <- ft_extract(pdf1, "xpdf"))
```

```
## <document>/Users/sacmac/Library/R/3.1/library/fulltext/examples/example1.pdf
##   Pages: 16
##   Title: ecsp-04-08-07 1..16
##   Producer: Acrobat Distiller 10.1.5 (Windows)
##   Creation date: 2013-08-16
```

Or extract directly into a `tm` Corpus


```r
paths <- sapply(paste0("example", 2:5, ".pdf"), function(x) system.file("examples", x, package = "fulltext"))
(corpus_rcamp <- ft_extract_corpus(paths, "rcamp"))
```


```r
## $meta
##   Author       DateTimeStamp Description            Heading ID Language
## 1     NA 2014-09-18 18:09:57          NA pone.0107412 1..10  1       en
## 2     NA 2014-09-15 15:10:27          NA pone.0106752 1..11  2       en
## 3     NA 2014-09-26 16:26:17          NA               <NA>  3       en
## 4     NA 2014-09-27 10:29:14          NA               <NA>  4       en
##   LocalMetaData                              Origin
## 1            NA 3B2 Total Publishing System 7.51n/W
## 2            NA 3B2 Total Publishing System 7.51n/W
## 3            NA        Adobe InDesign CS6 (Windows)
## 4            NA                                <NA>
##                                        class
## 1 PlainTextDocument, TextDocument, character
## 2 PlainTextDocument, TextDocument, character
## 3 PlainTextDocument, TextDocument, character
## 4 PlainTextDocument, TextDocument, character
##
## $data
## A corpus with 4 text documents
##
## attr(,"class")
## [1] "rcamp"
```

Remotely, using a service called PDFX


```r
pdf5 <- system.file("examples", "example5.pdf", package = "fulltext")
pdfx(file = pdf5)
```


```r
## $meta
## $meta$job
## [1] "34b281c10730b9e777de8a29b2dbdcc19f7d025c71afe9d674f3c5311a1f2044"
##
## $meta$base_name
## [1] "5kpp"
##
## $meta$doi
## [1] "10.7554/eLife.03640"
##
##
## $data
## <?xml version="1.0" encoding="UTF-8"?>
## <pdfx xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://pdfx.cs.man.ac.uk/static/article-schema.xsd">
##   <meta>
##     <job>34b281c10730b9e777de8a29b2dbdcc19f7d025c71afe9d674f3c5311a1f2044</job>
##     <base_name>5kpp</base_name>
##     <doi>10.7554/eLife.03640</doi>
##   </meta>
##    <article>
##  .....
```

## Meta

* Please [report any issues or bugs](https://github.com/ropensci/fulltext/issues).
* License: MIT
* This package is part of the [rOpenSci](http://ropensci.org/packages) project.
* Get citation information for `fulltext` in R doing `citation(package = 'fulltext')`

[![ropensci](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
