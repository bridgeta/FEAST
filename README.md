FEAST - a scalable algorithm for quantifying the origins of complex microbial communities
-----------------------

A major challenge of analyzing the compositional structure of microbiome data is identifying its potential origins. Here, we introduce Fast Expectation-mAximization microbial Source Tracking (*FEAST*), a ready-to-use scalable framework that can simultaneously estimate the contribution of thousands of potential source environments in a timely manner, thereby helping unravel the origins of complex microbial communities. The information gained from *FEAST* may provide insight into quantifying contamination, tracking the formation of developing microbial communities, as well as distinguishing and characterizing bacteria-related health conditions. 
For more details see Shenhav et al. 2019, Nature Methods (https://www.nature.com/articles/s41592-019-0431-x).

Support
-----------------------

For support using FEAST, please email: liashenhav@gmail.com


Software Requirements and dependencies
-----------------------

*FEAST* is implemented in R (>= 3.4.4) and requires the following dependencies: **Rcpp**, **RcppArmadillo**, **vegan**, **dplyr**, **reshape2**, **gridExtra**, **ggplot2**, **ggthemes**. Please install them prior to trying to install *FEAST*. If you are using a mac and having installation issues with **Rcpp** and or **RcppArmadillo**, try installing homebrew or xcode then reinstalling **Rcpp** and **RcppArmadillo**. 


Installation
---------------------------

*FEAST* will be available on Qiime II very soon. Until then you can you can simply install *FEAST* using **devtools**: 

```
devtools::install_github("cozygene/FEAST", ref = "FEAST_beta")
```

## Usage
As input, *FEAST* takes mandatory arguments:
- _C_ - An _m_ by _n_ count matrix, where _m_ is the number samples and _n_ is the number of taxa.
- _metadata_ - An _m_ by 3 table, where _m_ is the number samples. The metadata table has three colunms (i.e., 'Env', 'SourceSink', 'id'). The first column is a description of the sampled environment (e.g., human gut), the second column indicates if this sample is a source or a sink (can take the value 'Source' or 'Sink'). The fourth column is the Sink-Source id. When using multiple sinks, each tested with the same group of sources, only the rows with 'SourceSink' = Sink will get an id (between 1 - number of sinks in the data). In this scenatio, the sources ids are blank. When using multiple sinks, each tested with a distinct group of sources, each combination of sink and its corresponding sources should get the same id (between 1 - number of sinks in the data). Note that these names must be respected.
- _EM_iterations_ - A numeric value indicating the number of EM iterations (default 1000).
- _COVERAGE_ - A numeric value indicating the rarefaction depth (default = minimal sequencing depth within each group of sink and its corresponding sources).
- _different_sources_flag_ - A boolian value indicating the source-sink assignment. different_sources_flag = 1 if different sources are assigned to each sink , otherwise = 0.


*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1. Pij is the contribution of source j to sink i. If Pij == NA it indicateds that source j was not used in the analysis of sink i.



Demo
-----------------------
We provide a dataset for an example of FEAST's usage. Download the demo files <a href="https://github.com/cozygene/FEAST/tree/FEAST_beta/Data_files">here</a>.

First load the **FEAST** packages into R:
```
library(FEAST)
```

Then, load the datasets:
```
metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt")
otus <- Load_CountMatrix(CountMatrix_path = "~/FEAST/Data_files/otu_example_multi.txt")
```
Run _FEAST_, saving the output with prefix "demo":

```
FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, outfile="demo")
```

_FEAST_ will then save the file
*demo_FEAST.txt* - A file containing an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.


Input - 

Input format
-----------------------
The input to *FEAST* is composed of two tab-separated ASCII text files :

count table  - A matrix of samples by taxa with the sources and sink. The first row contains the sample ids ('SampleID'). The first column contains taxa ids. Then every consecutive column contains read counts for each sample. Note that this order must be respected (see example below).

count matrix (first 4 rows and columns):

| | ERR525698 |ERR525693 | ERR525688| ERR525699|
| ------------- | ------------- |------------- |------------- |------------- |
| taxa_1  |  0 | 5 | 0|20 |
| taxa_2  |  15 | 5 | 0|0 |
| taxa_3  |  0 | 13 | 200|0 |
| taxa_4  |  4 | 5 | 0|0 |


metadata -  The first row contains the headers ('SampleID', 'Env', 'SourceSink', 'id'). The first column contains the sample ids. The second column is a description of the sampled environment (e.g., human gut), the third column indicates if this sample is a source or a sink (can take the value 'Source' or 'Sink'). The fourth column is the Sink-Source id. 
When using multiple sinks, each tested with the same group of sources, only the rows with 'SourceSink' =  Sink will get an id (between 1 -  number of sinks in the data). In this scenatio, the sources ids are blank. 
When using multiple sinks, each tested with a distinct group of sources, each combination of sink and its corresponding sources should get the same id (between 1 -  number of sinks in the data). 
Note that these names must be respected  (see examples below).


*using multiple sinks, each tested with the same group of sources:

| SampleID | Env |SourceSink | id |
| ------------- | ------------- |------------- |-------------|
| ERR525698  |  infant gut 1 | Sink | 1
| ERR525693  |  infant gut 2 | Sink | 2 |
| ERR525688   |  Adult gut 1 | Source| NA |
| ERR525699  |  Adult gut 2 | Source | NA |
| ERR525697  |  Adult gut 3 | Source | NA |


*using multiple sinks, each tested with a different group of sources:

| SampleID | Env |SourceSink | id |
| ------------- | ------------- |------------- |-------------|
| ERR525698  |  infant gut 1 | Sink | 1
| ERR525688   |  Adult gut 1 | Source| 1 |
| ERR525691  |  Adult gut 2 | Source | 1 |
| ERR525699  |  infant gut 2 | Sink | 2 |
| ERR525697  |  Adult gut 3 | Source | 2 |
| ERR525696  |  Adult gut 4 | Source | 2 |


 

Output - 

| infant gut 2  |Adult gut 1 | Adult gut 2| Adult gut 3| Adult skin 1 |  Adult skin 2|  Adult skin 3| Soil 1 | Soil 2 | unknown|
| ------------- | ------------- |------------- |------------- |------------- |------------- |------------- |------------- |------------- |------------- |
|  5.108461e-01  |  9.584116e-23 | 4.980321e-12 | 2.623358e-02|5.043635e-13 | 8.213667e-59| 1.773058e-10 |  2.704118e-14 |  3.460067e-02 |  4.283196e-01 |




