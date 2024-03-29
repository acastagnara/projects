---
title: "Creating An Efficient Data Analysis Workflow"
author: "Alberto Castagnara"
date: "26/3/2021"
output:
  pdf_document: default
  html_document: default
---



# Introduction

In this guided project, we will be acting as a data analyst for a company that sells books for learning programming. Our company has produced multiple books, and each has received many reviews. Our company wants us to check out the sales data and see if we can extract any useful information from it. 

# Getting Familiar With The Data


```r
library(tidyverse)

## Loading the dataset
reviews <- read_csv('book_reviews.csv')

## Dimension of the dataset
dim(reviews)

## Column names
colnames(reviews)

## Types of each column
for (col in colnames(reviews)) {
  typeof(reviews[[col]])
}

## Unique values of each column
for (col in colnames(reviews)) {
  print("Unique values in the column:")
  print(col)
  print(unique(reviews[[col]]))
  print("")
}
```

# Handling Missing Data

Now that we are more familiar with the data itself, now we can get into the more specific details of data analysis. The first issue we will contend with is the issue of missing data. Missing data is annoying because there's nothing we can really do with it. We can't perform any analysis or calculations with missing data.

There are two ways that we can deal with missing data: 1) remove any rows or columns that have missing data (typically, rows) or 2) fill in the missing data in an informed, discipline way. This second way is known as imputation, and it's outside of the scope of what we've learned so far. For now, we'll take first approach with this dataset.


```r
complete_reviews <- reviews %>% 
  filter(!is.na(review))

dim(complete_reviews)
```

# Dealing With Inconsistent Labels

Now that we've removed all of the missing data from the dataset, we have a **complete** dataset. This is the ideal case that we would like to start any data analysis, so we're working towards a better dataset already.

The next thing that we need to work on is the `state` column. We noticed that the labeling for each state is inconsistent. For example, California is written as both "California" and "CA". Both "California" and "CA" refer to the same place in the United States, so we should try to clean this up. We need to choose one of the ways to refer to the state, and stick to that convention. Making labels/strings more consistent in the data will make things easier to analyze later on, so we'll handle this issue with this screen.


```r
complete_reviews <- complete_reviews %>%
  mutate(
    state = case_when(
      state == "California" ~ "CA",
      state == "New York" ~ "NY",
      state == "Texas" ~ "TX",
      state == "Florida" ~ "FL",
      TRUE ~ state # ignore cases where it's already postal code
    )
  )
```

# Transforming the Review Data

The first things we'll handle in the dataset are the reviews themselves. We noticed in our data exploration that the reviews take the form of strings, ranging from "Poor" to "Excellent". Our goal is to evaluate the ratings of each of the textbooks, but there's not much we can do with text versions of the review scores. It would be better if we were to convert the reviews into a numerical form.


```r
complete_reviews <- complete_reviews %>%
  mutate(
    review_num = case_when(
      review == 'Poor' ~ 1,
      review == 'Fair' ~ 2,
      review == 'Good' ~ 3,
      review == 'Great' ~ 4,
      review == 'Excellent' ~ 5),
    is_high_review = if_else(review_num >= 4, TRUE, FALSE)
)
head(complete_reviews)
```

# Analyzing the Data

It's important to keep the overall goal in mind as we handle all the little details of the cleaning. We are acting as an analyst trying to figure out which books are the most profitable for the company. The initial data wasn't in a form that was ready for analysis, so we needed to do this cleaning to prepare it. A lot of analysts starting their first jobs believe that the analysis part of their job will be the bulk of their work. To the contrary, a lot of our work will focus on data cleaning itself, while by comparison the data analysis might only take a few lines.

With all of our data cleaning, now we're ready to do some analysis of the data. Our main goal is to figure out what book is the most profitable. How will we judge what the "most profitable" book is though? Our dataset represents customer purchases. One way to define "most profitable" might be to just choose the book that's purchased the most. Another way to define it would be to see how much money each book generates overall.


```r
complete_reviews %>% 
  group_by(book) %>% 
  summarize(
    purchased = n()
  ) %>% 
  arrange(-purchased)
head(complete_reviews)
```

The books are relatively well matched in terms of purchasing, but "Fundamentals of R For Beginners" has a slight edge over everyone else. 

# Conclusions

In this guided project, we analyzed a dataset from a company to answer a business question they had. We explored the data, cleaned it and produced some valuable information for the book company in the form of a report. 
