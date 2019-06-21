# sc-interchange

Better interchange for single cell tools

## Summary of existing formats

### [`AnnData`](github.com/theislab/anndata) (`v0.7+`)

#### In memory

Based around two key dimensions, observations (`obs`, cells) and variables (`var`, genes). These are the the dimensions of the main expression matrix `X` and metadata about there are stored in a pair of dataframes `obs` and `var`.

Other matrices which are aligned to some set of the main axes are stored in mappings under the `layers`, `obsm`, `varm`, `obsp`, and `varp` attributes. These mappings can hold array-like objects (currently: arrays, sparse arrays, dataframes).

* `layers`: array-likes similar to `X`. The first dimension is aligned to the observations, and the second to the variables. Examples of values which go in here include different states of normalization (e.g. counts) and splice variant counts (for rna velocity).
* `obsm`: array-likes whose first dimension aligns with the observations. Values here include dimensionality reductions, or multimodal data.
* `varm`: array-likes whose first dimension correspond to the variables. Examples include the feature loadings of a PCA, or differential expression data.
* `obsp`: Pairwise measurements on the observations. E.g. adjacency matrices for cell-cell networks or distance matrices.
* `varp`: Pairwise measurements on the variables. Examples include gene-gene networks or correlations.

Anything which doesn't fit into these categories goes into the `uns` mapping.

#### On disk layout (`.h5ad`)

The on disk schema is broadly similar to the in memory one. The root group has keys for `X`, `layers`, `obsm`, `varm`, `obsp`, `varp`, and `uns`. Objects are represented as follows:

* Mappings are `Groups`
* Dense arrays are `Datasets`
* Record arrays are `Datasets` with compound data types
* Dataframes are arrays with compound dtypes. Categorical variables are encoded as hdf5 categorical data types. A dataframe must have an `index` column for the index/ row names. This is pretty close to how record arrays are stored.
* Sparse arrays are `Groups` containing a `Dataset` for each of the underlying arrays.

Each mapping is a group. We're starting to introduce conventions to idenitfy how each object should be decoded based on it's attrs, though this was previously done in an ad-hoc manner.

### [Loom](http://loompy.org)

**TODO**

### [SingleCellExperiment](https://bioconductor.org/packages/devel/bioc/html/SingleCellExperiment.html)

The `SingleCellExperiment` class is derived from the `SummarizedExperiment` class and thus shares its features:

* `assays`: a list of matrices where rows are typically features (e.g, genes) and columns are samples (e.g., cells).
Multiple matrices are allowed to accommodate different types of values, e.g., counts, RPMs, weights/offsets.
All matrices in a single `SummarizedExperiment` instance must have the same dimensions.
* `colData`: a data frame where each row corresponds to a column of a matrix in `assays`.
This is used to store sample-specific fields (e.g., treatment condition, cell type, library quality).
Any vector-like object can be stored in a single field, including nested data frames and matrices.
* `rowData`: a data frame where each row corresponds to a row of a matrix in `assays`.
This is used to store feature-specific fields (e.g., gene synonyms).
Any vector-like object can be stored in a single field, including nested data frames and matrices.
* `rowRanges`: a vector of genomic intervals where each entry corresponds to a row of a matrix in `assays`.
This is used to store genomic coordinate information for each feature (e.g., locations of genes, SNPs, peaks).
* `metadata`: a list of arbitrary objects containing general metadata about the object.
This is traditionally used to store information about the entire study or experiment.

The `SingleCellExperiment` provides the following additional features:

* `reducedDims`: a list of matrices with the same number of rows, where each row corresponds to a column of a matrix in `assays`.
Each matrix corresponds to a single dimensionality result result (from t-SNE, UMAP, PCA, etc.).
Different results can have variable numbers of columns.
* `int_colData`: same as `colData`, but internal.
It is intended for developer use and should not be accessed by users.
* `int_rowData`: same as `rowData`, but internal.
It is intended for developer use and should not be accessed by users.

These aspects are summarized in the figure below:

![](https://osca.bioconductor.org/images/singlecellexperiment.png )

The `SingleCellExperiment` class itself has no mandated on-disk format.
Individual matrices in `assays` or fields in the `colData`/`rowData` may be file-backed objects (e.g., `HDF5Matrix`s),
but the choice of representation and file format is left to the discretion of the user.
