Prerequisite
------------

To preprocess scFlowExamples dataset, the following packages should be
installed.

``` r
if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}
BiocManager::install("DropletUtils")

install.packages("ids")

devtools::install_github("NathanSkene/One2One")

devtools::install_github(repo = "hhoeflin/hdf5r")

devtools::install_github(repo = "mojaveazure/loomR", ref = "develop")
```

Install the dataset repository
------------------------------

``` r
devtools::install_github("neurogenomics/scFlowExample")
```

Setup the dataset
-----------------

The current dataset uses `TEINH15`, `TEINH19`, `MGL1`, `MOL1` cells. For
a list of available cell types please visit [this
link](http://mousebrain.org/celltypes/). Use the following codes only if
you want to use different cell types.

``` r
# create a temporary directory
td <- tempdir()

# create the placeholder file
tf <- tempfile(tmpdir = td, fileext = ".loom")

# download into the placeholder file
download.file("https://storage.googleapis.com/linnarsson-lab-loom/l5_all.loom", tf) # tf="~/l5_all.loom"
unzip(tf)

allExp <- prep_zeisel2018(path = tf)
keptExp <- merge_zeisel_celltypes(allExp, useCells = c("TEINH15", "TEINH19", "MGL1", "MOL1"))
indvExp <- split_celltypes_byIndv(keptExp, joinCells = c("TEINH15", "TEINH19"), 
                                  nCases = 3, jointName = "TEINH")

# Save dataset so that it can be used easily
usethis::use_data(indvExp, overwrite = TRUE)
```

Downsampling the dataset
------------------------

The user can downsample the dataset by reducing the cell number and gene
number using the following commands. To downsample the dataset use the
`keptExp` object created in the previous step.

#### To downsample cells only

``` r
keptExp_ds <- downsample_cells(keptExp = keptExp,
                               prop_cell = c(0.5,0.5,0.05,0.02))

indvExp_ds <- split_celltypes_byIndv(keptExp_ds, joinCells = c("TEINH15", "TEINH19"), 
                                              nCases = 3, jointName = "TEINH")

# Save dataset so that it can be used easily
usethis::use_data(indvExp_ds, overwrite = TRUE)
```

#### To downsample both cells and genes

``` r
keptExp_ds_4K <- downsample_cells(keptExp = keptExp,
                                    prop_cell = c(0.5,0.5,0.05,0.02),
                                    n_top_genes = 4000)
indvExp_ds_4K <- split_celltypes_byIndv(keptExp_ds_4K, joinCells = c("TEINH15", "TEINH19"), 
                                                        nCases = 3, jointName = "TEINH")

# Save dataset so that it can be used easily
usethis::use_data(indvExp_ds_4K, overwrite = TRUE)
```

Save the data to file
---------------------

You could just run the following codes and continue from here. The
following codes will generate scFlowExample dataset in 10x genomics
Cellranger output format, a `Manifest.txt` file containing data path for
individual samples and a `SampleSheet.tsv` containing sample metadata.

``` r
library(scFlowExamples)

#To use the full size dataset
data("indvExp", package = "scFlowExamples")  

#To use a downsampled dataset
data("indvExp_ds", package = "scFlowExamples") #This dataset contains all genes (~29000)

#To use a downsampled dataset with 4000 genes
data("indvExp_ds_4K", package = "scFlowExamples") 

#To write out the data in 10X genomics format
write_data(indvExp, output_dir = "full/path/to/output/dir")
write_scflow_manifest(indvExp, output_dir = "full/path/to/output/dir")
write_scflow_samplesheet(indvExp, output_dir = "full/path/to/output/dir")
```
