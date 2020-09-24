Lab 06 - Text Mining
================
Brandyn Ruiz
9/23/2020

# Learning goals

  - Use `unnest_tokens()` and `unnest_ngrams()` to extract tokens and
    ngrams from text.
  - Use dplyr and ggplot2 to analyze text data

# Lab description

For this lab we will be working with a new dataset. The dataset contains
transcription samples from <https://www.mtsamples.com/>. And is loaded
and “fairly” cleaned at
<https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv>.

This markdown document should be rendered using `github_document`
document.

# Setup the Git project and the GitHub repository

1.  Go to your documents (or wherever you are planning to store the
    data) in your computer, and create a folder for this project, for
    example, “PM566-labs”

2.  In that folder, save [this
    template](https://raw.githubusercontent.com/USCbiostats/PM566/master/content/assignment/06-lab.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository, hopefully of
    the same name that this folder has, i.e., “PM566-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

### Setup packages

You should load in `dplyr`, (or `data.table` if you want to work that
way), `ggplot2` and `tidytext`. If you don’t already have `tidytext`
then you can install with

``` r
#install.packages("tidytext")
```

``` r
library(dplyr)
library(ggplot2)
library(readr)
library(tidytext)
library(tidyr)
```

### read in Medical Transcriptions

Loading in reference transcription samples from
<https://www.mtsamples.com/>

``` r
mt_samples <- read_csv("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv")
mt_samples <- mt_samples %>%
  select(description, medical_specialty, transcription)

head(mt_samples)
```

    ## # A tibble: 6 x 3
    ##   description                  medical_specialty   transcription                
    ##   <chr>                        <chr>               <chr>                        
    ## 1 A 23-year-old white female ~ Allergy / Immunolo~ "SUBJECTIVE:,  This 23-year-~
    ## 2 Consult for laparoscopic ga~ Bariatrics          "PAST MEDICAL HISTORY:, He h~
    ## 3 Consult for laparoscopic ga~ Bariatrics          "HISTORY OF PRESENT ILLNESS:~
    ## 4 2-D M-Mode. Doppler.         Cardiovascular / P~ "2-D M-MODE: , ,1.  Left atr~
    ## 5 2-D Echocardiogram           Cardiovascular / P~ "1.  The left ventricular ca~
    ## 6 Morbid obesity.  Laparoscop~ Bariatrics          "PREOPERATIVE DIAGNOSIS: , M~

``` r
mt_samples$transcription[1]
```

    ## [1] "SUBJECTIVE:,  This 23-year-old white female presents with complaint of allergies.  She used to have allergies when she lived in Seattle but she thinks they are worse here.  In the past, she has tried Claritin, and Zyrtec.  Both worked for short time but then seemed to lose effectiveness.  She has used Allegra also.  She used that last summer and she began using it again two weeks ago.  It does not appear to be working very well.  She has used over-the-counter sprays but no prescription nasal sprays.  She does have asthma but doest not require daily medication for this and does not think it is flaring up.,MEDICATIONS: , Her only medication currently is Ortho Tri-Cyclen and the Allegra.,ALLERGIES: , She has no known medicine allergies.,OBJECTIVE:,Vitals:  Weight was 130 pounds and blood pressure 124/78.,HEENT:  Her throat was mildly erythematous without exudate.  Nasal mucosa was erythematous and swollen.  Only clear drainage was seen.  TMs were clear.,Neck:  Supple without adenopathy.,Lungs:  Clear.,ASSESSMENT:,  Allergic rhinitis.,PLAN:,1.  She will try Zyrtec instead of Allegra again.  Another option will be to use loratadine.  She does not think she has prescription coverage so that might be cheaper.,2.  Samples of Nasonex two sprays in each nostril given for three weeks.  A prescription was written as well."

-----

## Question 1: What specialties do we have?

We can use `count()` from `dplyr` to figure out how many different
catagories do we have? Are these catagories related? overlapping? evenly
distributed?

``` r
mt_samples %>%
  count(medical_specialty, sort = TRUE)
```

    ## # A tibble: 40 x 2
    ##    medical_specialty                 n
    ##    <chr>                         <int>
    ##  1 Surgery                        1103
    ##  2 Consult - History and Phy.      516
    ##  3 Cardiovascular / Pulmonary      372
    ##  4 Orthopedic                      355
    ##  5 Radiology                       273
    ##  6 General Medicine                259
    ##  7 Gastroenterology                230
    ##  8 Neurology                       223
    ##  9 SOAP / Chart / Progress Notes   166
    ## 10 Obstetrics / Gynecology         160
    ## # ... with 30 more rows

These categories are all related in a medical facility like a hospital.
The categories do not overlap with each other. However, the categories
are not evenly distributed as Surgery appeared the most with 1103
appearances compared to Autopsy that only has a frequency of 8.

-----

## Question 2

  - Tokenize the the words in the `transcription` column
  - Count the number of times each token appears
  - Visualize the top 20 most frequent words

Explain what we see from this result. Does it makes sense? What insights
(if any) do we get?

``` r
library(forcats)
```

    ## Warning: package 'forcats' was built under R version 3.6.3

``` r
mt_samples%>%
  unnest_tokens(token, transcription)%>%
  count(token)%>%
  top_n(20, n)%>%
  ggplot(aes(x = n, y = fct_reorder(token, n)))+
  geom_col()+
  labs(title = 'Frequency of Tokens', x = 'N', y = 'Token')
```

![](Lab6_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

From the result we see a lot of stop words being the most frequent
words. The most frequent being the word ‘the’ having a frequency of
150,000.

-----

## Question 3

  - Redo visualization but remove stopwords before
  - Bonus points if you remove numbers as well

What do we see know that we have removed stop words? Does it give us a
better idea of what the text is about?

``` r
# stop_words

mt_samples%>%
  unnest_tokens(token, transcription)%>%
  anti_join(stop_words, by = c('token' = 'word'))%>%
  filter(!(token %in% as.character(seq(0, 100))))%>%
  count(token)%>%
  top_n(20, n)%>%
  ggplot(aes(x = n, y = fct_reorder(token, n)))+
  geom_col()+
  labs(title = 'Frequency of Tokens', x = 'N', y = 'Token', caption = 'Stop words and Numbers removed')
```

![](Lab6_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

When removing the stop words we have a better idea of what the text is
about because we see now that patient is the most used word with a
frequency being 20,000. There are alos other medical terminologies like
procedure, history, and pain for example.

-----

# Question 4

repeat question 2, but this time tokenize into bi-grams. how does the
result change if you look at tri-grams?

``` r
mt_samples %>%
  unnest_ngrams(ngram, transcription, n = 2)%>%
  count(ngram, sort = TRUE)%>%
  top_n(20, n)%>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n)))+
  geom_col()+
  labs(title = 'Frequency of Bi-grams', x = 'N', y = 'Bi-gram')
```

![](Lab6_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
mt_samples %>%
  unnest_ngrams(ngram, transcription, n = 3)%>%
  count(ngram, sort = TRUE)%>%
  top_n(20, n)%>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n)))+
  geom_col()+
  labs(title = 'Frequency of Tri-grams', x = 'N', y = 'Tri-gram')
```

![](Lab6_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

-----

# Question 5

Using the results you got from questions 4. Pick a word and count the
words that appears after and before it.

``` r
trigram <- mt_samples%>%
  unnest_ngrams(ngram, transcription, n = 3)%>%
  separate(ngram, into = c('word1', 'word2', 'word3'), sep = ' ')%>%
  select(word1, word2, word3)

#count of words before 'patient'
trigram%>%
  filter(word2 == 'patient')%>%
  anti_join(stop_words, by = c('word1' = 'word'))%>%
  count(word1, sort = TRUE)
```

    ## # A tibble: 220 x 2
    ##    word1           n
    ##    <chr>       <int>
    ##  1 history       101
    ##  2 procedure      32
    ##  3 female         26
    ##  4 sample         23
    ##  5 male           22
    ##  6 illness        16
    ##  7 plan           16
    ##  8 indications    15
    ##  9 allergies      14
    ## 10 correct        11
    ## # ... with 210 more rows

``` r
#count of words after 'patient'
trigram%>%
  filter(word2 == 'patient')%>%
  anti_join(stop_words, by = c('word2' = 'word'))%>%
  count(word3, sort = TRUE)
```

    ## # A tibble: 588 x 2
    ##    word3         n
    ##    <chr>     <int>
    ##  1 was        6289
    ##  2 is         3330
    ##  3 has        1417
    ##  4 tolerated   992
    ##  5 had         886
    ##  6 will        616
    ##  7 denies      552
    ##  8 and         377
    ##  9 states      363
    ## 10 does        334
    ## # ... with 578 more rows

``` r
#Frequency table of words before and after 'Patient'
trigram%>%
  filter(word2 == 'patient')%>%
  anti_join(stop_words, by = c('word1' = 'word'))%>%
  anti_join(stop_words, by = c('word2' = 'word'))%>%
  count(word1, word3, sort = TRUE)
```

    ## # A tibble: 301 x 3
    ##    word1       word3      n
    ##    <chr>       <chr>  <int>
    ##  1 history     admits    37
    ##  2 history     is        25
    ##  3 procedure   was       20
    ##  4 history     denies    18
    ##  5 allergies   admits    14
    ##  6 illness     is        13
    ##  7 female      who       12
    ##  8 indications is        10
    ##  9 lbs         is         9
    ## 10 exam        is         8
    ## # ... with 291 more rows

-----

# Question 6

Which words are most used in each of the specialties. you can use
`group_by()` and `top_n()` from `dplyr` to have the calculations be done
within each specialty. Remember to remove stopwords. How about the most
5 used words?

``` r
#The 5 most used words for each medical speciality
mt_samples%>%
  unnest_tokens(token, transcription)%>%
  anti_join(stop_words, by = c('token' = 'word'))%>%
  filter(!(token %in% as.character(seq(0, 100))))%>%
  group_by(medical_specialty)%>%
  count(token)%>%
  top_n(5, n)
```

    ## # A tibble: 210 x 3
    ## # Groups:   medical_specialty [40]
    ##    medical_specialty    token         n
    ##    <chr>                <chr>     <int>
    ##  1 Allergy / Immunology allergies    21
    ##  2 Allergy / Immunology history      38
    ##  3 Allergy / Immunology nasal        13
    ##  4 Allergy / Immunology noted        23
    ##  5 Allergy / Immunology past         13
    ##  6 Allergy / Immunology patient      22
    ##  7 Autopsy              anterior     47
    ##  8 Autopsy              body         40
    ##  9 Autopsy              inch         59
    ## 10 Autopsy              left         83
    ## # ... with 200 more rows

# Question 7 - extra

Find your own insight in the data:

Ideas:

  - Interesting ngrams
  - See if certain words are used more in some specialties then others
