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
data <- read_csv(file = "./data/data_clustered_continuous.csv")
```

    ## Rows: 8 Columns: 8

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (1): id
    ## dbl (7): treatment, n_group, n_cluster, mean1, sd1, mean2, sd2

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
  mutate(n_effective = n_group/DE) %>%
  mutate(n_effective = round(n_effective, digits = 0))
```

# Calculating a correlation coefficient

In this example, we will use corr = 0.5

``` r
corr <- 0.5
```

# Calculate change and SD

``` r
data <- data %>%
  mutate(change = mean2 - mean1,
         change_sd = sqrt(sd1^2 + sd2^2 - 2*corr*sd1*sd2)) %>%
  mutate(change_sd = round(change_sd, digits = 2))
```

# Finalize the dataset

``` r
data_final_continuous <- data %>%
  select(id, treatment, n_effective, change, change_sd) %>%
  mutate(n1 = if_else(treatment == 1, n_effective, 0),
         mean1 = if_else(treatment == 1, change, 0),
         sd1 = if_else(treatment == 1, change_sd, 0),
         n2 = if_else(treatment == 0, n_effective, 0),
         mean2 = if_else(treatment == 0, change, 0),
         sd2 = if_else(treatment == 0, change_sd, 0)) %>%
  group_by(id) %>%
  mutate(n2 = sum(n2),
         mean2 = sum(mean2),
         sd2 = sum(sd2)) %>%
  filter(treatment == 1) %>%
  select(-treatment, -n_effective, -change, -change_sd)
```

``` r
write_xlsx(data_final_continuous, path = "./data/data_final_continuous.xlsx")
```
