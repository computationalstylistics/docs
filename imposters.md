Authorship verification with the package `stylo`
================
Maciej Eder
30/05/2018

## Introduction

NA

The implementation provided by the package `stylo` is a rather faithful
interpretation of two algorithms described in a study on authorship
verification (Kestemont et al., 2016). A tiny function for computing the
score `c@1` used to evaluate the system is directly transplanted from
the original implementation
(<https://github.com/mikekestemont/ruzicka>).

NA

NA

## Installation

The latest (and stable) version of the package `stylo` is usually
available on CRAN a few days after such a new version is released. In
this case, the installation is trivial:

``` r
install.pacgages("stylo")
```

NA

``` r
library(devtools)
install_github("computationalstylistics/stylo")
```

NA

## A tl;dr working example

NA

``` r
# activating the package 'stylo':
library(stylo)

# activating one of the datasets provided by the package 'stylo';
# this is a table of frequences of a few novels, including "The Cuckoo's Calling"
# by Robert Galbraith, aka JK Rowling:
data(galbraith)

# to learn more about the dataset, type:
help(galbraith)

# to see the table itself, type:
galbraith

# now, time for launching the imposters method:
imposters(galbraith)
```

After a few seconds, the final results will be shown on the
    screen:

    ## 

    ## No candidate set specified; testing the following classes (one at a time):

    ##   coben   lewis   rowling   tolkien

    ## 

    ## Testing a given candidate against imposters...

    ## coben     0.31

    ## lewis     0

    ## rowling   1

    ## tolkien   0

    ##   coben   lewis rowling tolkien 
    ##    0.31    0.00    1.00    0.00

NA

## Details

NA

Despite simplicity, however, this solution is far from being flexible.
In a vast majority of cases, one would like to have some control on
choosing the text to be contrasted against the corpus. The function
provides a dedicated parameter `test` to do the trick. Note the
following
code:

``` r
# getting the 8th row from the dataset (it contains frequencies for Galbraith):
my_text_to_be_tested = galbraith[8,]

# building the reference set so that it does not contain the 8th row
my_frequency_table = galbraith[-c(8),]

# launching the imposters method:
imposters(reference.set = my_frequency_table, test = my_text_to_be_tested)
```

NA

``` r
my_text_to_be_tested = galbraith[24,]
my_frequency_table = galbraith[-c(24),]
imposters(reference.set = my_frequency_table, test = my_text_to_be_tested)
```

By the way, it might be non trivial to know in advance which row of the
input table contains your disputed text. The simplest way to get the
content of the table is to request its row names via `rownames()`. One
can also use `grep()` to identify a given string of characters e.g.:

``` r
# getting the names of the texts
rownames(galbraith)

# getting the row number of a particular text (known by name):
grep("lewis_lion", rownames(galbraith)) 

# one can also combine the above-introduced snippets into one piece:
text_name = grep("lewis_lion", rownames(galbraith))
my_text_to_be_tested = galbraith[text_name,]
my_frequency_table = galbraith[-c(text_name),]
imposters(reference.set = my_frequency_table, test = my_text_to_be_tested)
```

NA

``` r
# indicating the text to be tested (here, "The cuckoo's Calling"):
my_text_to_be_tested = galbraith[8,]

# defining the texts by the candidate author (here, the texts by JK Rowling):
my_candidate = galbraith[16:23,]

# building the reference set by excluding the already-selected rows
my_imposters = galbraith[-c(8, 16:23),]

# launching the imposters method:
imposters(reference.set = my_imposters, test = my_text_to_be_tested, candidate.set = my_candidate)
```

NA

## Loading a corpus from text files

NA

``` r
# activating the package
library(stylo)

# setting a working directory that contains the corpus, e.g.
setwd("/Users/m/Desktop/A_Small_Collection_of_British_Fiction/corpus")

# loading the files from a specified directory:
tokenized.texts = load.corpus.and.parse(files = "all")

# computing a list of most frequent words (trimmed to top 2000 items):
features = make.frequency.list(tokenized.texts, head = 2000)

# producing a table of relative frequencies:
data = make.table.of.frequencies(tokenized.texts, features, relative = TRUE)

# who wrote "Pride and Prejudice"? (in my case, this is the 4th row in the table):
imposters(reference.set = data[-c(4),], test = data[4,])
```

One important remark to be made, is that the frequency table is analyzed
in its entirety. In the above example, the input vector of features
(most frequent words) has 2000 elements. If you want to run the
`imposters()` function on a shorter vector of words, you should select
them in advance, e.g. to get 100 most frequent words, type:

``` r
imposters(reference.set = data[-c(4), 1:100], test = data[4, 1:100])
```

## Optimizing the decision scores

NA

NA

``` r
# activating another dataset, which contains Southern American novels:
data(lee)

# getting some more information about the dataset
help(lee)

# running the computationally-intense optimalization
imposters.optimize(lee)
```

NA

    ## [1] 0.43 0.55

NA

NA

## Parameters

NA

``` r
# activating the package 'stylo':
library(stylo)

# activating one of the datasets provided by the package 'stylo':
data(galbraith)

# Classic Delta distance
imposters(galbraith, distance = "delta")

# Cosine Delta (aka Wurzburg Distance)
imposters(galbraith, distance = "wurzburg")

# Ruzicka Distance (aka Minmax Distance)
# (please keep in mind that it takes AGES to compute it!)
imposters(galbraith, distance = "minmax")
```

NA

``` r
# activating the package 'stylo':
library(stylo)

# activating another dataset, which contains Southern American novels:
data(lee)

# defining the test text, i.e. "In Cold Blood"
my_text_to_be_tested = lee[1,]

# defining the comparison corpus
my_reference_set = lee[-c(1),]

# NOW, time to test 4 different distance measures:

# Classic Delta distance
imposters(my_reference_set, my_text_to_be_tested, distance = "delta")

# Eder's Delta distance
imposters(my_reference_set, my_text_to_be_tested, distance = "eder")

# Cosine Delta (aka Wurzburg Distance)
imposters(my_reference_set, my_text_to_be_tested, distance = "wurzburg")

# Ruzicka Distance (aka Minmax Distance)
# (please keep in mind that it takes AGES to compute it!)
imposters(my_reference_set, my_text_to_be_tested, distance = "minmax")
```

NA

Other parameters of the function `imposters()` include:

NA \* `features` (default: 0.5) indicates the share of features
(e.g. words) to be randomly picked in each iteration. If the feature
vector has 500 most frequent words, and the `features` parameter is set
to the value 0.1, then in each iteration a random subset of 10% of the
words (i.e. 50) is selected. NA

Some other, more techical, parameters can be found in the manual page of
the function. Type `help(imposters)` for the details.

Certainly, the same applied to the `imposters.optimize()` function. The
same parameters that were introduced immediately above, can be passed to
the fine-tuning function, e.g.:

``` r
results = imposters(my_reference_set, distance = "wurzburg")

results1 = imposters(my_reference_set, distance = "wurzburg", imposters = 0.8)
```

Try all of them\!

## References

<div id="refs" class="references">

<div id="ref-kestemont_authorship_2016">

**Kestemont, M., Stover, J., Koppel, M., Karsdorp, F. and Daelemans,
W.** (2016). Authorship verification with the Ruzicka metric. In,
*Digital Humanities 2016: Conference Abstracts*. Kraków: Jagiellonian
University & Pedagogical University, pp. 246–49
<http://dh2016.adho.org/abstracts/402>.

</div>

</div>
