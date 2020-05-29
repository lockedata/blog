---
slug: distribute-data
title: How to distribute data with your R package
authors: 
- Maëlle Salmon
date: '2020-05-29'
tags:
- package development
- standards
- data
output: hugodown::hugo_document
rmd_hash: 349ab31ff64914d7

---




Distributing data with an R package can be crucial for the package or even the only goal of a package:
to show what a function can accomplish with a dataset; to show how a package can help tidy a messy data format; to test the package; for teaching purposes; to allow users to directly use the bundled data instead of having to fetch and clean the data.
Now, _how_ to provide data with/for your package is a recurring theme in [R package development channels](/2019/04/11/r-package-devel/).
In this post, we shall present various ways to distribute data with/for an R package, depending on the data use case and on its size.

_Thanks to the R connoisseurs [Thomas Vroylandt](https://tvroylandt.netlify.app/), [Sébastien Rochette](https://statnmap.com/) and [Hugo Gruson](https://www.normalesup.org/~hgruson/) for providing some inspiration and resources for this post! :pray: [^grrr]_

## Data in your package

Sometimes the data can be *vendored*[^vendor] i.e. live in your package source, and even built into and installed with the package.
An excellent overview of the different cases is provided in the [R packages book by Hadley Wickham and Jenny Bryan](https://r-pkgs.org/data.html); without forgetting the reference ["Writing R Extensions"](https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Data-in-packages).
In `usethis`, as explained in the R packages book, there are [helpers for creating package data](https://usethis.r-lib.org/reference/use_data.html).

### Data for whom?

* If the data is for the user to load and use in examples or their own code, you're looking for [exported data](https://r-pkgs.org/data.html#data-data).
Since it's exported, it has to be documented. 
For an example, see the [`babynames` `data/` folder and `R/data.R` file](https://github.com/hadley/babynames/)

* If the data is for your functions to use internally, you're after `R/sysdata.R` or an internal function.

    * `R/sysdata.R`. E.g. that's where `mimetypes` are stored in the [`magick` packages](https://github.com/ropensci/magick/blob/c116b2b8505f491db72a139b61cd543b7a2ce873/tools/mimetypes.R)
    * An [internal function](/2019/12/12/internal-functions/). E.g. how do you store the languages your pluralize and singularize functions support? It could be an internal function whose advantage is to be readable (as opposed to seeing a sysdata.rda in a repo) and whose downside is that it might be less natural to [generate it with code](/2020/02/10/code-generation/). Example:

```r
my_languages <- function() {
  c("en", "fr")
}
# elsewhere
blabla code my_languages() blabla code
```

* If the data is to showcase how to tidy raw data, and you want the user to be able to access it, you're after... [raw data living in inst/extdata](https://r-pkgs.org/data.html#data-extdata).

* If you want to keep the raw data used to create your exported or internal data, which you should, you're after the `data-raw` folder.
See [the `data-raw` folder of the `babynames` package](https://github.com/hadley/babynames/tree/master/data-raw), in particular its R scripts.
Note that `data-raw` has to be in the [`.Rbuildignore` file](/2020/05/20/rbuildignore/) (but `usethis` would help you with that).

* For data used in test, i.e. fixtures, you could create a folder under e.g. `tests/testthat/` and whose content would be found using the [`testthat::test_path()`](https://testthat.r-lib.org/reference/test_path.html) function.

### Data as small as possible

As "Writing R Extensions" underlines, for data under `data/` and, _"If your package is to be distributed, do consider the resource implications of large datasets for your users: they can make packages very slow to download and use up unwelcome amounts of storage space, as well as taking many seconds to load."_.
So what do you do?

* You can _compress_ the data, internal or external. Your friends are `tools::resaveRdaFiles()`

```r 
?tools::resaveRdaFiles
Report on Details of Saved Images or Re-saves them
```

* You can refer the next section of this post and have data live outside of your package!

For data you use in tests, i.e. fixtures, you can think about how to make it as small as possible whilst still providing enough bug discovery potential.
In other words, save minimal test cases, not a ton of data.
This will make your package source lighter.

## Data outside of your package

Now, sometimes the data is too big to be in your package, or follows a different development cycle i.e. is updated more or less often.
What can you do about that?
Well, have data live somewhere else.

### Data packages

Yes, this subsection makes the point of having data live inside _another_ package.  :expressionless:
You could develop _companion_ packages to go with your package, that hold data.
A data package is user-friendly in the sense that installing it saves the data on the machine and makes it available.
This is the setup of [`rnaturalearth`](https://github.com/ropensci/rnaturalearth), for which one of the companion packages, [`rnaturalearthhires`](https://github.com/ropensci/rnaturalearthhires), is not hosted on CRAN.


{{< tweet 1262759210946682888 >}}

For a clear explanation of a way to host data packages outside of CRAN, refer to the R Journal article by Brooke Anderson and Dirk Eddelbuettel, ["Hosting Data Packages via drat: A Case Study with Hurricane Exposure Data"](https://journal.r-project.org/archive/2017/RJ-2017-026/index.html).

You could also check out the [`datastorr` package](https://docs.ropensci.org/datastorr/) (not on CRAN) for integrating data packages with GitHub.

### Other data services

Not using a data package also helps you make the data available e.g. as CSV to anyone including, gasp, Python users. :snake:

* You could use an existing infrastructure: GitHub releases (with the [`piggyback` package](https://docs.ropensci.org/piggyback/) whose documentation includes a [thorough comparison with other approaches such as git-LFS](https://docs.ropensci.org/piggyback/articles/alternatives.html)), Amazon S3 (using the [`pins` package](http://pins.rstudio.com/)?), etc.[^cb]

* You could... write your [own web API](/2019/06/10/cran-checks-api/)? Like the web APIs powering R-hub's own `pkgsearch`, `rversions`. The web APIs have value of their own, and you can write wrapper functions in your package that thus becomes a data access package.

This is all good so far but with non data packages your package should still help download and save the data for a seamless workflow, e.g. via a function.
What can this function do with the data?
It could of course returns the data as an object, but also save it locally for easier re-use in future analyses.

* The data could be cached in an app dir which we explained [in a previous blog post](/2020/03/12/user-preferences/), using the `rappdirs` or `hoardr` package;

* For other approaches, your package could take a dependency on a package aimed at data storage and retrieval such as the aforementioned `piggyback` and `pins` packages, the [`bowerbird` package](https://docs.ropensci.org/bowerbird/), the [`rdataretriever` package](https://docs.ropensci.org/rdataretriever/).

## Conclusion

In this post we went over different setups allowing you to distribute data with or for your R package: some standard locations in your package source (`inst/extdata/`, `data/`, `R/sysdata.rda`); locations outside of your package (using `drat`, `git-LFS`, GitHub releases via `piggyback`, a web API you'd build yourself), that your package would know how to access, and potentially save locally (in an app dir, or using a tool like `bowerbird`).
Do _you_ have a "data and R package" setup you'd like to share?
How do you document the data you share, when it's bigger than a small table?[^dataspice]
Please comment below!

[^grrr]: In a conversation in the friendly French-speaking R Slack workspace -- where we'd write _connaisseurs_, not _connoisseurs_. If you want to join us, follow [the invitation link](https://github.com/frrrenchies/frrrenchies#cat-chat-et-discussions-instantan%C3%A9es-cat). _À bientôt !_
[^vendor]: Now that you know the word *vendor*, [_" to bundle one's own, possibly modified version of dependencies with a standard program."_](https://www.wordhippo.com/what-is/the-verb-for/vendor.html), you can use your search engine to find and enjoy debates around vendoring or not. You're welcome.
[^dataspice]: E.g. the [`dataspice` package](https://github.com/ropenscilabs/dataspice) aims at creating lightweight schema.org descriptions of datasets.
[^cb]: Thanks to [Carl Boettiger](https://www.carlboettiger.info/) for useful insight, including reminding me of the `pins` package.