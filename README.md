# ReproducibleScience


This is a toolbox to start a reproducible science project that clearly
separates raw data, a model, and the views, reports, and presentations. Such organization takes advantage of several tools available in an R package, such as automated documentation, testing, and graphing. Prerequisites are a basic knowledge of R language, otherwise, take [the R Introduction Course](https://swcarpentry.github.io/r-novice-gapminder/).


# Tutorial

This is a tutorial on how to get started with R packages used specifically towards the goal of reproducible science.

# Advantages

Generally, several team members contribute to a science project; sometimes a model evolves or is refined over time, and a transparent track of generating a final graph is needed. Imagine the following scenario. Team member 1 fabricates a sample, and then makes a measurement and records digital data from an instrument, team member 2 takes data for the same samples with a different instrument. All three data sets are considered `raw data` and are stored in its original form.

Team member 1 wants to analyze the data and make a graph for a presentation. They create an R package and then writes a function that returns all data files that are needed for the analysis. Then the member writes a model, it takes the name of a data file and outputs a fitting parameter, a graph, or table. Finally, they write a function that creates the graph for the presentation; it can be saved. 

Team member 2 also needs to make a presentation, and writes their own R package, but reuses the model from team member 1 to generate the same data set by invoking team member 1's library and its functions, which are well documented.






### 1. R Package

In the first step, you will [generate a Git Repository](https://docs.github.com/en/desktop/installing-and-configuring-github-desktop/overview/creating-your-first-repository-using-github-desktop) - usually private for version control - and create a [new R Package](https://hub.packtpub.com/how-to-create-your-own-r-package-with-rstudio-tutorial/). To do so, make use of two packages `devtools` and `roxygen2`.

``` r
install.packages("roxygen2")
install.packages("devtools")
```



### 2. Setup of Basic Files

Next, modify the `DESCRIPTION` file, and create some additional files using the `testthat` package:

``` r
library(usethis)
usethis::use_readme_md()
usethis::use_news_md()
```

Add a license, choosing the appropriate license, here are two examples:

``` r
usethis::use_gpl3_license()
usethis::use_proprietary_license('Thomas Gredig')
```

For documentation, run the [roxygen2 command](https://roxygen2.r-lib.org/articles/roxygen2.html), you may have to delete the file `NAMESPACE` before running it.

``` r
roxygen2::roxygenise()
```

Also, add `PackageRoxygenize: rd,collate,namespace` to the .Rproj file to execute `devtools::document()` from the Build menu.



