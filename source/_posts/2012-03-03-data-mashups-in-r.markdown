---
layout: post
title: "Data Mashups in R"
date: 2012-03-03 20:31
comments: true
categories: Book Reading
---

## Ch.I Mapping Foreclosures ##

- download
    
        download.file(url="URL", destfile="DEST")

- regex

``` R
grep()
gsub()
```

- Yahoo!'s Latitude and Longitude service: [sign up](http://developer.yahoo.com/wsregapp)

- Parse XML

``` R
install.packages("XML")
library("XML")
xmlTreeParse(requestURL, isURL=T)

install.package("RCurl")
library("RCurl")
```

- Proxy

        Sys.setenv("http_proxy" = "http://username:passwd@host:port")

- *Magic* `str()`
    
    it is good practice to closely examine each package's data structures using `str()`

- Exception handling

``` R
tryCatch({
    xmlResult <- xmlTreeParse(requestURL, isURL=TRUE, addAttributeNamespaces=TRUE)
    #...other code...
    }, error=function(err){
        cat("xml parsing or http error:", conditionMessage(err), "\n")
})
```
- [PBSmapping](http://cran.r-project.org/web/packages/PBSmapping/index.html)

``` R
library(PBSmapping)
plotPolys()
```

- Exploring Data Structures

``` R
as.numeric()
level()
```

- Color

``` R
head.colors()
```
---
## Ch.II Statistics of Foreclosure ##

- Data is available at [FactFinder](http://factfinder.census.gov/servlet/DCGeoSelectServlet?ds_name=DEC_2000_SF3_U)

- skip lines when reading from file

``` R
read.table("FILE", skip=1, na.string="")
```
- Descriptive Statistics: `mean()`, `median()`, `sd()`, `cor()`, `summary()`.

- lattice

        library(lattice)
        install.packages(latticsExtra)
        library(latticeExtra)

    - plot: `stripplot()` + `bwplot()`

``` R
    print(stripplot(IncomeLevels ~ jitter(ct$FCs),
        main=list(
        "Foreclures grouped by National Median Household Income", cex=1),
        sub=list("Greater or less than $50,000", cex=1),
        xlab="foreclosures",
        ylab="household median income",
        aspect=.3, col="lightblue", pch=2 ) + 
    as.layer(bwplot(IncomeLevels ~ ct$FCs, varwidth=TRUE, box.ratio=0.4, col="blue", pch="|"))
    )
```

- Correlation

    In R, we can create multidimensional correlation graphs using the `pairs()` scatterplot matrix package.
    

---
#†#
