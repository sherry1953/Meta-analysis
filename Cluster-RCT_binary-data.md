Cluster-RCT
================
Xinyan Zhou
10/27/2021

This is a tool for using effective sample sizes to do approximate
analyses for cluster- RCT.

If some of our included studies are cluster-RCT, we cannot direct
analyze data from these studies like we do for individually randomized
trials. We need to do approximate analyses to consider the cluster
design.

# Input the data

You can input your data either use import dataset below “Enviroment” or
use the following code. If you would like to use the following code, you
should first upload your data (.csv) into the “data” file.

Remember: Please format your data file the same as the example data set!

``` r
data <- read_csv(file = "./data/data_clustered_binary.csv")
```

    ## Rows: 6 Columns: 6

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (1): id
    ## dbl (5): treatment, n_group, n_cluster, n1, n0

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

# Decide the intracluster correlation coefficient

You can change the value of ICC here.

``` r
ICC <- 0.02
```

# Calculate the average cluster size, M

``` r
data <-
  data %>%
  group_by(id) %>%
  mutate(n_total = sum(n_group),
         n_cluster_total = sum(n_cluster))

data <- 
  data %>%
  mutate(M = n_total/n_cluster_total)
```

# Calculate the design effect, DE

``` r
data <- 
  data %>%
  mutate(DE = 1 + (M - 1)*ICC)
```

# Calculate the effective sample sizes

``` r
data <-
  data %>%
  mutate(n1 = n1/DE, n0 = n0/DE) %>%
  mutate(n1 = round(n1, digits = 0),
         n0 = round(n0, digits = 0))
```

# Finalize the dataset

``` r
data_final_binary <- data %>%
  select(id, treatment, n1, n0) %>%
  mutate(n11 = if_else(treatment == 1, n1, 0),
         n12 = if_else(treatment == 1, n0, 0),
         n21 = if_else(treatment == 0, n1, 0),
         n22 = if_else(treatment == 0, n0, 0)) %>%
  group_by(id) %>%
  mutate(n21 = sum(n21),
         n22 = sum(n22)) %>%
  filter(treatment == 1) %>%
  select(-treatment, -n1, -n0)
```

``` r
write_xlsx(data_final_binary, path = "./data/data_final_binary.xlsx")
```
