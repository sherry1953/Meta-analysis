Continuous data
================
Xinyan Zhou
2021/10/27

# Input the data

You can input your data either use import dataset below “Enviroment” or
use the following code. If you would like to use the following code, you
should first upload your data (.csv) into the “data” file.

Remember: Please format your data file the same as the example data set!

``` r
data <- read_csv(file = "./data/data_continuous.csv")
```

    ## Rows: 4 Columns: 7

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (1): id
    ## dbl (6): treatment, mean1, sd1, mean2, sd2, n

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

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
data_final <- data %>%
  select(id, treatment, n, change, change_sd) %>%
  mutate(n1 = if_else(treatment == 1, n, 0),
         mean1 = if_else(treatment == 1, change, 0),
         sd1 = if_else(treatment == 1, change_sd, 0),
         n2 = if_else(treatment == 0, n, 0),
         mean2 = if_else(treatment == 0, change, 0),
         sd2 = if_else(treatment == 0, change_sd, 0)) %>%
  group_by(id) %>%
  mutate(n2 = sum(n2),
         mean2 = sum(mean2),
         sd2 = sum(sd2)) %>%
  filter(treatment == 1) %>%
  select(-treatment, -n, -change, -change_sd)
```

``` r
write_xlsx(data_final, path = "./data/data_final.xlsx")
```
