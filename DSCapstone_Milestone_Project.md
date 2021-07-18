---
title: "<span style='font-size: 48px'>Data Science Capstone - Milestone Report</span>"
author: "<span style='font-size: 24px'>Henry CY Wong</span>"
date: "<span style='font-size: 24px'>2021-07-18</span>"
output:
    html_document:
            css: style.css
            keep_md: true
---

&nbsp;
&nbsp;



## 1. Introduction

This milestone report is used to illustrate the exploratory data analysis of the given SwiftKey dataset as a preparation of the Shiny App and algorithm of the Capstone Project. The corresponding source code is shown in the Appendix section for reference.

&nbsp;
&nbsp;

## 2. Libraries Used

The following libraries will be used:


```r
library(stringi)
library(tm)
library(RWeka)
library(ggplot2)
```

&nbsp;
&nbsp;


## 3. Data Load

The dataset will be downloaded from [https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip](https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip).
Three data files included in the downloaded dataset which are

- **en_US.blogs.txt**
- **en_US.news.txt**
- **en_US.twitter.txt** 





The summary of given files is as shown below:

```
##           file_name file_size_in_Mb lines_per_file words_per_file
## 1   en_US.blogs.txt        200.4242         899288       37570839
## 2    en_US.news.txt        196.2775        1010242       34494539
## 3 en_US.twitter.txt        159.3641        2360148       30451170
```

&nbsp;
&nbsp;

## 4. Data Preparation and Cleaning

Since the size of data is huge, in order to reduce the processing time, a smaller dataset is considered. Hence, 1% of the number of lines per file will be extracted as samples for analysis. Extracted samples from three given files will be combined into one sample dataset for cleaning and analysis. The number of the sample file after extraction and consolidation is


```
## [1] 42695
```

Then, the sample dataset will be converted into a corpus for the following data cleaning:

* Remove URL
* Remove Twitter Handler
* Remove Email Address
* Remove Profanity Words
* Transform all words to lowercase
* Remove English Stop Words
* Remove Numbers
* Remove Punctuation

**Credit: The Profanity Word List is downloaded from [Luis von Ahn's](https://www.cs.cmu.edu/~biglou/resources/) Research Group for corresponding profanity checking and removal.**





The sample after cleaning will look like:


```
##                                                                                                                                            text
## 1  whats next  ales  autism  april  autism awareness month greg   couple events   works hes  ready  discuss yet      can  sure ill let  know   
## 2                                                                                                                  wouldnt  opened  anyone else
## 3                                                   finish   flourless chocolate cake ethereally light  fluffy   nightcap  pedro ximenez sherry
## 4                                                                                   ways people can verify  ’m saying  publicrecord information
## 5                                                                              youve grown older  interests  narrowed books dolls girlie things
## 6                                                                                                                                  india factor
```

&nbsp;
&nbsp;

## 5. Exploratory Data Analysis

Since the objective of the Capstone Project is used to create a N-gram predictive model to predict the next word according to the word(s)/phrase(s) of users input, therefore the Unigram, Bigram and Trigarm will be tokenized from the sample dataset for analysis.

&nbsp;

### 5.1 Unigram

![](DSCapstone_Milestone_Project_files/figure-html/show_unigram-1.png)<!-- -->

&nbsp;

### 5.2 Bigram

![](DSCapstone_Milestone_Project_files/figure-html/show_bigram-1.png)<!-- -->

&nbsp;

### 5.3 Trigram

![](DSCapstone_Milestone_Project_files/figure-html/show_trigram-1.png)<!-- -->

&nbsp;
&nbsp;

## 6. Next Steps

The next steps will 

- build a N-gram Prediction Model according to the analysis of this Milestone Report; and
- build a Data Product with using Shiny to predict the next word according to user input.

&nbsp;
&nbsp;

## 7. Appendix

Below are the source code of this report for reference.

&nbsp;
&nbsp;

### 7.1 Data Load


```r
data_url <- "https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip"
data_file <- "Coursera-SwiftKey.zip"
data_dir <- "./data"

if (!file.exists(data_dir)) {
    dir.create(data_dir)
    }

if (!file.exists(data_file)) {
    download.file(data_url, destfile = data_file, method = "curl")
    unzip(data_file, exdir = data_dir)
}

blogs_file <- "./data/final/en_US/en_US.blogs.txt"
news_file <- "./data/final/en_US/en_US.news.txt"
twitter_file <- "./data/final/en_US/en_US.twitter.txt" 

# read files
blogs <- readLines(blogs_file, encoding = "UTF-8", skipNul = TRUE)
news <- readLines(news_file, encoding = "UTF-8", skipNul = TRUE)
twitter <- readLines(twitter_file, encoding = "UTF-8", skipNul = TRUE)

# file sizes in Mb
blogs_size <- file.size(blogs_file)/(1024^2)
news_size <- file.size(news_file)/(1024^2)
twitter_size <- file.size(twitter_file)/(1024^2)

# number of lines per files
blogs_lines <- length(blogs)
news_lines <- length(news)
twitter_lines <- length(twitter)

# words per files
blogs_words <- stri_stats_latex(blogs)[4]
news_words <- stri_stats_latex(news)[4]
twitter_words <- stri_stats_latex(twitter)[4]

# file summary
file_summary <- data.frame(file_name = c("en_US.blogs.txt", "en_US.news.txt", "en_US.twitter.txt"),
                           file_size_in_Mb = c(blogs_size, news_size, twitter_size),
                           lines_per_file = c(blogs_lines, news_lines, twitter_lines),
                           words_per_file = c(blogs_words, news_words, twitter_words))
#file_summary
```

&nbsp;
&nbsp;

### 7.2 Data Preparation and Cleaning


```r
sample_size <- 0.05
set.seed(20210718)
blogs_samples <- sample(blogs, size = blogs_lines * sample_size, replace = TRUE)
news_samples <- sample(news, size = news_lines * sample_size, replace = TRUE)
twitter_samples <- sample(twitter, size = twitter_lines * sample_size, replace = TRUE)
my_samples <- c(blogs_samples, news_samples, twitter_samples)
#length(my_samples)

# load profanity dataset
profanity_url <- "https://www.cs.cmu.edu/~biglou/resources/bad-words.txt"
profanity_file <- "./data/bad-words.txt"
data_dir <- "./data"

if (!file.exists(data_dir)) {
    dir.create(data_dir)
    }

if (!file.exists(profanity_file)) {
    download.file(profanity_url, destfile = profanity_file, method = "curl")
    }

profanity_words <- readLines(profanity_file, encoding = "UTF-8", skipNul = TRUE)

# data cleaning

my_corpus <- VCorpus(VectorSource(my_samples))

to_space <- content_transformer(function(in_str, my_pattern) gsub(my_pattern, " ", in_str))

url_pattern <- "(f|ht)tp(s?)://(.*)[.][a-z]+"
twitter_pattern <- "@[^\\s]+"
email_pattern <- "\\b[A-Z a-z 0-9._ - ]*[@](.*?)[.]{1,3} \\b"

my_corpus <- tm_map(my_corpus, to_space, url_pattern)
my_corpus <- tm_map(my_corpus, to_space, twitter_pattern)
my_corpus <- tm_map(my_corpus, to_space, email_pattern)

my_corpus <- tm_map(my_corpus, removeWords, profanity_words)
my_corpus <- tm_map(my_corpus, tolower)
my_corpus <- tm_map(my_corpus, removePunctuation)
my_corpus <- tm_map(my_corpus, removeNumbers)
my_corpus <- tm_map(my_corpus, stripWhitespace)
my_corpus <- tm_map(my_corpus, removeWords, stopwords("en"))
my_corpus <- tm_map(my_corpus, PlainTextDocument)

# convert corpus to data frame
my_corpus_df <- data.frame(text = unlist(sapply(my_corpus, '[', "content")), stringsAsFactors = FALSE)

#head(my_corpus_df)
```

&nbsp;
&nbsp;

### 7.3 Exploratory Data Analysis

&nbsp;

#### 7.3.1 Unigram


```r
unigram <- NGramTokenizer(my_corpus_df, Weka_control(min = 1, max = 1))
unigram <- data.frame(table(unigram))
unigram <- unigram[order(unigram$Freq, decreasing = TRUE),]
names(unigram) <- c("one_word", "freq")

g_unigram <- ggplot(data = unigram[1:20,], aes(x=reorder(one_word, freq), y=freq))
g_unigram <- g_unigram + geom_bar(stat="identity") + coord_flip()
g_unigram <- g_unigram + xlab("") + ylab("Frequency") + ggtitle("Top 20 Unigrams")
#g_unigram
```

&nbsp;

#### 7.3.2 Bigram


```r
bigram <- NGramTokenizer(my_corpus_df, Weka_control(min = 2, max = 2))
bigram <- data.frame(table(bigram))
bigram <- bigram[order(bigram$Freq, decreasing = TRUE),]
names(bigram) <- c("two_words", "freq")

g_bigram <- ggplot(data = bigram[1:20,], aes(x=reorder(two_words, freq), y=freq))
g_bigram <- g_bigram + geom_bar(stat="identity") + coord_flip()
g_bigram <- g_bigram + xlab("") + ylab("Frequency") + ggtitle("Top 20 Bigrams")
#g_bigram
```

&nbsp;

#### 7.3.3 Trigram


```r
trigram <- NGramTokenizer(my_corpus_df, Weka_control(min = 3, max = 3))
trigram <- data.frame(table(trigram))
trigram <- trigram[order(trigram$Freq, decreasing = TRUE),]
names(trigram) <- c("three_words", "freq")

g_trigram <- ggplot(data = trigram[1:20,], aes(x=reorder(three_words, freq), y=freq))
g_trigram <- g_trigram + geom_bar(stat="identity") + coord_flip()
g_trigram <- g_trigram + xlab("") + ylab("Frequency") + ggtitle("Top 20 Trigrams")
#g_trigram
```

&nbsp;
&nbsp;
