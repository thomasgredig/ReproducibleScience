# ReproducibleScience


This is a toolbox to start a reproducible science project that clearly
separates raw data, a model, and the views, reports, and presentations. Such organization takes advantage of several tools available in an R package, such as automated documentation, testing, and graphing. Prerequisites are a basic knowledge of R language, otherwise, take [the R Introduction Course](https://swcarpentry.github.io/r-novice-gapminder/).

You will need to create two packages: a **data R package**, and an **analysis R package**. If your project name is `highHR`, you would create two packages, called `dataHighHR` (raw data) and `highHR` (analysis, graphs).

Data from multiple instruments (XRD, AFM, etc.) are processed into one or several data sets; these data sets have common columns with information that is processed in the in `dataHighHR` package only. On contrary, the analysis portion, does not import data files, but only uses data sets from memory. These data sets are loaded into memory with:

```{r}
library(dataHighHR)
```



# Tutorial

This is a tutorial on how to get started with R packages used specifically towards the goal of reproducible science.

## Advantages

Generally, several team members contribute to a science project; sometimes a model evolves or is refined over time, and a transparent track of generating a final graph is needed. Imagine the following scenario. Team member 1 fabricates a sample, and then makes a measurement and records digital data from an instrument, team member 2 takes data for the same samples with a different instrument. All three data sets are considered `raw data` and are stored in its original form.

Team member 1 wants to analyze the data and make a graph for a presentation. They create an R package and then writes a function that returns all data files that are needed for the analysis. Then the member writes a model, it takes the name of a data file and outputs a fitting parameter, a graph, or table. Finally, they write a function that creates the graph for the presentation; it can be saved.

Team member 2 also needs to make a presentation, and writes their own R package, but reuses the model from team member 1 to generate the same data set by invoking team member 1's library and its functions, which are well documented.






###  R Package

In the first step, you will [generate a Git Repository](https://docs.github.com/en/desktop/installing-and-configuring-github-desktop/overview/creating-your-first-repository-using-github-desktop) - usually private for version control - and create a [new R Package](https://hub.packtpub.com/how-to-create-your-own-r-package-with-rstudio-tutorial/). To do so, make use of two packages `devtools` and `roxygen2`.

``` r
install.packages("roxygen2")
install.packages("devtools")
```

# Data Package

In the following, we will build a data package with the `RAW data files`.


### Setup 

After you created a new package, modify some files:

Modify the `DESCRIPTION` file, and create some additional files using the `testthat` package:

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
file.remove('NAMESPACE')
roxygen2::roxygenise()
```

Also, add `PackageRoxygenize: rd,collate,namespace` to the .Rproj file to execute `devtools::document()` from the Build menu. (Note: if "Document" does not appear, then go to `Build -> Configure Build Tools` and enable it.)



### RAW data files

Create a folder for the RAW data and initialize some other stuff:

``` r
usethis::use_data_raw()
```


Create unique IDs for each file, so that if you change the file name, or move them to another folder later on, you can still find them via the **unique ID**, which identifies a file and recognizes it based on the MD5 checksum.

``` r
# devtools::install_github("thomasgredig/RAWdataR")
library(RAWdataR)
raw.updateID(verbose=TRUE)
```

Create a folder `data-raw` and move all RAW files into that folder. You can arrange them in separate sub-folders, such as `AFM/TG20220101Si1-FePc150nm`, referring to a particular sample with AFM data.

### Create Dataset

Create a data set; in this example we will show how to add AFM files, for example, since this requires generating two data sets, first create the `make.dataAFM.R` file, which is responsible of curating the data.

``` r
file.create('data-raw/make.dataAFM.R')
```

Edit this file and add two portions, similar to this example code:

``` r
library(dplyr)
library(nanoscopeAFM)
library(RAWdataR)
verbose = TRUE

# 1. Find Image Files
# ===========================
raw.updateID()
dataRAW %>% filter(type=='AFM' & missing==FALSE) -> rFile
if (verbose) print(paste("Found: ", nrow(rFile), "AFM images."))

getDirection <- function(filename) {
  s = ""
  if (grepl('Backward', filename)) s= "Backward"
  if (grepl('Forward', filename)) s = "Forward"
  s
}


# 2. Make Data File Table
# ==========================
result = data.frame()
for(j in 1:nrow(rFile) ) {
  fname = file.path(rFile$path[j], rFile$filename[j])
  d = AFM.import(fname)

  result = rbind(result, data.frame(
    ID = rFile$ID[j] ,
    sample = rFile$sample[j],
    direction = getDirection(rFile$filename[j]),
    imageNo = as.numeric(gsub('.*\\_(\\d{3})\\..*','\\1', rFile$filename[j])),
    summary(d)
  ))
}
result$history <- NULL

# 3. Select high quality images
# ==========================
afmIDs = c(8, 12, 15)
result$quality = ""
result$quality[which(result$ID %in% afmIDs)] = "high"

# 4. Save Data Table to DB
# ==========================
dataFilesAFM = result
usethis::use_data(dataFilesAFM, overwrite = TRUE)


# 5. Gather AFM data
# ==========================
dataRAW %>% filter(ID %in% afmIDs) -> rFile
afmd = list()
for(j in 1:nrow(rFile) ) {
  fname = file.path(rFile$path[j], rFile$filename[j])
  d = AFM.import(fname)
  afmd[[rFile$ID[j]]] = AFM.selectChannel(d, 1)
}


# 6. Save AFM files to DB
# ==========================
dataAFM = afmd
usethis::use_data(dataAFM, overwrite = TRUE)
```

Now, it is time to document the data set, you can use the following function from the `sinew` R package to generate a template for the documentation. 

This documentation goes in the `R` folder, into a file called `DATASETS.R` or similar.

``` r
sinew::makeOxygen('dataFilesAFM', add_fields = 'source')
sinew::makeOxygen('dataAFM', add_fields = 'source')
```

The output would look something like this:
``` r
#' @title AFM files data set
#' @description List of files and their attributes that contain AFM images
#' @format A data frame with 152 rows and 12 variables:
#' \describe{
#'   \item{\code{ID}}{integer unique ID}
#'   \item{\code{sample}}{character sample name}
#'   \item{\code{direction}}{character direction of scanning}
#'   \item{\code{imageNo}}{double image number during scan}
#'   \item{\code{objectect}}{character instrument}
#'   \item{\code{description}}{character description of mode, NCM = non-contact mode}
#'   \item{\code{resolution}}{character resolution in pixels}
#'   \item{\code{size}}{character size of the images with units}
#'   \item{\code{channel}}{character channel name}
#'   \item{\code{z.min}}{double minimum value}
#'   \item{\code{z.max}}{double maximum value}
#'   \item{\code{z.units}}{character units for min and max value}
#'   \item{\code{quality}}{character "high" quality images are available in dataAFMList}
#'}
"dataFilesAFM"
```

Now, you can load the data and also create the help (`man`) files, by running the following sequence, the `load_all()` function will run all `R` files and load those functions into memory

``` r
devtools::document()
devtools::load_all(".")
? dataFilesAFM
```

Go to the `Build` menu and run **"Install and Restart"**; then "build source package".

At this moment, the data should have been saved in the `data` folder; this folder is generated every time you run `make.dataAFM.R` script. 

Since you will have many RAW data files, it is important to manage those files using `raw.updateID()` from the RAWdataR package. You can also get the ID from a filename useing `raw.getIDfromFile()` and conversely, given a filename, you can obtain the ID, using `raw.getFilefromID()`. It is worthwhile to save it as a separate data set as follows:

``` r
## code to prepare RAW_ID data set
library(dplyr)
library(RAWdataR)
verbose = TRUE

raw.updateID('data-raw','data-raw', verbose=TRUE)
dataRAW <- read.csv('data-raw/RAW-ID.csv')
dataRAW$path = gsub('.*\\/RAW/','',dataRAW$path)
dataRAW$sample = gsub('.*([A-Z][A-Z]\\d{6}[A-Z]*).*','\\1', dataRAW$filename)

m = grep('png$', dataRAW$filename)
dataRAW$type[m] = 'image'

m = grep('\\.R$', dataRAW$filename)
dataRAW$type[m] = 'R'

m = grep('\\.xlsx$', dataRAW$filename)
dataRAW$type[m] = 'Excel'

m = grep('^RAW-ID', dataRAW$filename)
dataRAW$type[m] = 'RAW-ID'

if (verbose) names(dataRAW)

usethis::use_data(dataRAW, overwrite = TRUE)
```

At this moment, keep adding RAW data files and create separate datasets. It maybe useful to document some of the data sets and its purpose in the README.MD file as well. It is good practice to backup the data set on GitHub, everytime you add more source data, you would then increase the version number and that would allow you to go back to a particular version to reproducibly regenerate your graphs and results. 


# Analysis Package

Now, that you have a data package; it can be updated, whenever there is new data that is needed. It should be curated, so that it is easy to load data with the analysis package; all exceptions and comments must be included in the *Data Package*, but no analysis should be done in that package.

Create a new pacakge, called **highHR**, in this case. This package is strictly dedicated to the analysis and it does not have to worry about folders, file locations, etc. All data is available from the data file.

Before using the data package, you may need to *Restart R*, to load the datasets back into memory. Alternatively, after building the dataset, you can also install it on a different computer, using the `install_local()` function; this will **install the R data package**.

``` r
devtools::install_local("dataHighHR_0.1.4.tar.gz")
```

Next, you can test it by loading and displaying and AFM image:

``` r
library(dplyr)
library(dataHighHR)

dataFilesAFM %>% filter(quality=='high') -> rFile
print(rFile)
ID1 = rFile$ID[1]
plot(dataAFM[[ID1]])
```

At this point, you can write some functions in the analysis package and move them to the `R` folder. Also, you can create vignettes with test cases. You can generate a template for a vignette with:

``` r
usethis::use_vignette("AFM-Images")
```

An example is given and the `pkgdown` documentation tool will evaluate this example and also show the result, unless you explicity state that you don't want to run it with `if (FALSE) {}`. Since the function makes use of functions from other packages, those need to be declared with `@importFrom`, after running `devtools::document()`, the function is automatically added to the `NAMESPACE` file (if you run it the first time, you may need to delete `NAMESPACE`, so it can be generated with the `document()` function; this function will also create the files in the `man` folder.)


For more information see [R Package Metadata](https://r-pkgs.org/description.html).

After adding a new function, you can load all functions in the `R` folder with:

``` r
devtools::load_all(".")
```

You have to make sure that there is no test code in the `R` folder, so I recommend to make a `_main.R` file in the main folder, where you can test some of the code, once it works, commit it as a function and documentation in the `R` folder.



### 3. Documentation

Use the `pkgdown` package to automatically render documentation for all functions. Prerequisite is that you provide a description and example code before each function.

``` r
pkgdown::build_site(lazy=TRUE)
```


### 4. Testing

Automated code testing is a good standard. First, set up a test environment:

``` r
usethis::use_testthat()
```

Next, create a test function, for example "general", this will automatically create a file with an example.

``` r
usethis::use_test('general')
```
