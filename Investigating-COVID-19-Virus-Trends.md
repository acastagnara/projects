Investigating COVID-19 Virus Trends
================
Alberto Castagnara
25/3/2021

## Introduction

A pneumonia of unknown cause detected in Wuhan, China was first
internationally reported from China on 31 December 2019. Today we know
this virus as Coronavirus. COVID-19 which stands for **CO**rona**VI**rus
**D**isease is the disease caused by this virus. Since then, the world
has been engaged in the fight against this pandemic. Several measures
have therefore been taken to “flatten the curve”. We have consequently
experienced social distancing and many people have passed away as well.

In the solidarity to face this unprecedented global crisis, several
organizations did not hesitate to share several datasets allowing the
conduction of several kinds of analysis in order to understand this
pandemic.

It is natural for us to analyze these datasets by ourselves to answer
questions since we cannot always rely on the news, and we are data
scientists.

In this Guided Project, we use [a dataset, from
Kaggle](https://www.kaggle.com/lin0li/covid19testing), that we have
prepared and made [available here for
download](https://dq-content.s3.amazonaws.com/505/covid19.csv). This
dataset was collected between the 20th of January and the 1st of June
2020. The purpose of this Guided Project is to build our skills and
understanding of the data analysis workflow by evaluating the COVID-19
situation through this dataset. At the end of this project, feel free to
download the updated version of the dataset and take the same steps to
analyze it.

Our analysis tries to provide an answer to this question: Which
countries have had the highest number of positive cases against the
number of tests?

## Understanding the Data

``` r
library(readr)

## Reading the dataframe
covid_df <- read.csv('covid19.csv')

## Determine the dimension of the dataframe
dimensions <- dim(covid_df)

##Determine the column names of covid_df
vector_names <- colnames(covid_df)
print(vector_names)

## Display the first few rows
head(covid_df)

## Display the summary
glimpse(covid_df)
```

## Isolating the Rows We Need

At this step, we continue our work with **dataframes**. Our goal is to
extract the data that is relevant to answer our questions.

As we were saying at the previous step, there might be some columns in
our dataset with inconsistencies. Looking at the few lines of our
dataset we displayed in the previous step, we can see that the
`Province_State` column mixes data from different levels: country level
and state/province level. Since we cannot run an analysis on all these
levels at the same time, we need to filter what we are interested in.

We will, therefore, extract only the country-level data in order to not
bias our analyses. To do so, we filter the data to keep only the data
related to `"All States"`. `"All States"` represents the value of the
column `Province_State` to specify that the COVID-19 data is only
available at the country level.

``` r
covid_df_all_states <- covid_df %>% filter(Province_State == "All States")
head(covid_df_all_states)
print(dim(covid_df_all_states))
```

## Isolating the Columns We Need

At this step, we continue our work with **dataframes**. Our goal is to
extract the data that is relevant to answer our questions.

Revisiting the description of the dataset columns, we can notice that
there are columns that provide daily information and others that provide
cumulative information.

1.  `Date`: Date
2.  `Continent_Name`: Continent names
3.  `Two_Letter_Country_Code`: Country codes
4.  `Country_Region`: Country names
5.  `Province_State`: States/province names; value is `All States` when
    state/provincial level data is not available
6.  `positive`: Cumulative number of positive cases reported.
7.  `active`: Number of actively cases on that **day**.
8.  `hospitalized`: Cumulative number of hospitalized cases reported.
9.  `hospitalizedCurr`: Number of actively hospitalized cases on that
    **day**.
10. `recovered`: Cumulative number of recovered cases reported.
11. `death`: Cumulative number of deaths reported.
12. `total_tested`: Cumulative number of tests conducted.
13. `daily_tested`: Number of tests conducted on the **day**; if daily
    data is unavailable, daily tested is averaged across number of days
    in between.
14. `daily_positive`: Number of positive cases reported on the **day**;
    if daily data is unavailable, daily positive is averaged across
    number of days in.

Hence, we should manage those cases (columns with cumulative and daily
information) separately because we cannot work with both together.
Actually, our analysis would be biased if we made the mistake of
comparing a column containing cumulative data and another one containing
only one-day data. This is another example of a situation that we want
to know from the beginning of the project in order to better analyze our
dataset.

Thereafter, we work mainly with daily data. So let’s extract the columns
related to the daily measures.

``` r
covid_df_all_states_daily <- covid_df_all_states %>%  select(Date, Country_Region, active, hospitalizedCurr, daily_tested, daily_positive)
head(covid_df_all_states_daily)
print(dim(covid_df_all_states_daily))
```

## Extraxting the Top Ten Tested Cases Countries

Our goal here is to extract the **top ten cases countries data.**

``` r
covid_df_all_states_daily_sum <- covid_df_all_states_daily %>% 
  group_by(Country_Region) %>%
  summarize(
    tested = sum(daily_tested),
    positive = sum(daily_positive),
    active = sum(active),
    hospitalized = sum(hospitalizedCurr)
  ) %>%
  arrange(-tested)

covid_df_all_states_daily_sum

## Extracting the top ten
covid_top_10 <- head(covid_df_all_states_daily_sum, 10)
covid_top_10
```

## Identifying the Highest Positive Against Tested Cases

Our goal is to answer this question: **Which countries have had the
highest number of positive cases against the number of tests?**

``` r
## Getting vectors
countries <- covid_top_10 %>% pull(Country_Region)
tested_cases <- covid_top_10 %>% pull(tested)
positive_cases <- covid_top_10 %>% pull(positive)
active_cases <- covid_top_10 %>% pull(active)
hospitalized_cases <- covid_top_10 %>% pull(hospitalized)

## Naming vectors
names(positive_cases) <- countries
names(tested_cases) <- countries
names(active_cases) <- countries
names(hospitalized_cases) <- countries

## Identify the top three positive against tested cases

print(positive_cases / tested_cases)
positive_tested_top_3 <- c("United Kingdom" = 0.11, "United States" = 0.10, "Turkey" = 0.08)
```

## Keeping Relevant Information

Our goal is to find a way to keep all the information available for the
top three countries that have had the highest number of positive cases
against the number of tests.

The previous step allowed identifying those top three countries as:

-   United Kingdom: 0.11
-   United States: 0.10
-   Turkey: 0.08

To make sure we won’t lose other information about these countries we
can create a matrix that contains the ratio and the overall number of
COVID-19 tested, positive, active and hospitalized cases.

``` r
## Creating vectors
united_kingdom <- c(0.11, 1473672, 166909, 0, 0)
united_states <- c(0.10, 17282363, 1877179, 0, 0)
turkey <- c(0.08, 2031192, 163941, 2980960, 0)

## Creating a matrix
covid_mat <- rbind(united_kingdom, united_states, turkey)
colnames(covid_mat) <- c("Ratio", "tested", "positive", "active", "hospitalized")
covid_mat
```

## Putting All Together

Our goal is to put all our answers and datasets together. Since a list
can contain several types of objects, we are able to store all the data
of our project together. This allows us to have a global view from a
single variable and the ability to export our results for other uses.

On the previous steps we answered the following questions:

-   Which countries have had the highest number of deaths due to
    COVID-19?
-   Which countries have had the highest number of positive cases
    against the number of tests?

Our answers are stored in the variables `positive_tested_top_3`.

To do so, we created several datastructures such as:

-   Dataframes: `covid_df`, `covid_df_all_states`,
    `covid_df_all_states_daily`, and `covid_top_10`.
-   Matrix: `covid_mat`.
-   Vectors: `vector_cols` and `countries`.

Let’s create a list to store all our work in the same variable.

``` r
question <- "Which countries have had the highest number of positive cases against the number of tests?"
answer <- c("Positive tested cases" = positive_tested_top_3)

datasets <- list(
  original = covid_df,
  allstates = covid_df_all_states,
  daily = covid_df_all_states_daily,
  top_10 = covid_top_10
)

matrices <- list(covid_mat)
vectors <- list(vector_names, countries)
data_structure_list <- list("dataframe" = datasets, "matrix" = matrices, "vector" = vectors)
covid_analysis_list <- list(question, answer, data_structure_list)
covid_analysis_list[[2]]
```
