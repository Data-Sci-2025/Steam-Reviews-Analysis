# 3-classifier


**Note**: Please run
[0-data-exploration](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/0-data-exploration.qmd),
[1-data-cleanup](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/1-data-cleanup.qmd),
and
[2-data-analysis](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.qmd)
first to create the version of the .csv file needed to start here.

## Classifier Data

To get another look at the df to be used to train the classifier later.
This is the tf-idf information for the downsampled version of the game
reviews data, stop words removed.

``` r
review_tf_idf <- read_csv("../private/reviews_tfidf.csv", show_col_types = FALSE)

review_tf_idf
```

    # A tibble: 1,241,994 × 5
       review_id review_type word          total tf_idf
           <dbl> <chr>       <chr>         <dbl>  <dbl>
     1 105382110 POS         hide             25  0.493
     2 105382110 POS         aesthetic        25  0.239
     3 105382110 POS         basically        25  0.169
     4 105382110 POS         beat             25  0.182
     5 105382110 POS         button           25  0.191
     6 105382110 POS         cool             25  0.148
     7 105382110 POS         essentially      25  0.216
     8 105382110 POS         haunted          25  0.283
     9 105382110 POS         house            25  0.197
    10 105382110 POS         interactivity    25  0.283
    # ℹ 1,241,984 more rows

### Remove NA

Just to double check and clean out any troublesome NA values:

``` r
colSums(is.na(review_tf_idf))
```

      review_id review_type        word       total      tf_idf 
              0           0        4231        4231        4231 

``` r
review_tf_idf <- na.omit(review_tf_idf)
```

``` r
colSums(is.na(review_tf_idf))
```

      review_id review_type        word       total      tf_idf 
              0           0           0           0           0 

``` r
review_tf_idf |>
  group_by(review_type) |>
  summarise(count=n())
```

    # A tibble: 2 × 2
      review_type  count
      <chr>        <int>
    1 NEG         793792
    2 POS         443971

The difference in data here is not necessarily *massive*, but did
initially surprise me! The is, until I ran the calculations in this
notebook
[here](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#late-data-overview).
There are fewer rows marked POS in this smaller dataset because of what
we already know: positive reviews are shorter than negative reviews.
Simply, there are fewer words marked POS overall.

## One more cleanup

I removed highly repetitive words at the end of my analysis notebook
that appeared very commonly in both positive and negative reviews. I
wanted to do this before tf-idf was calculated so that the weights of
each word would be calculated accurately.

To further encourage the classifier to be the best that it can be, I
chose to select for words with a higher TF-IDF value to select for the
more unique (and therefore more indicative) words.

``` r
reviews_shortened <- review_tf_idf |>
  filter(tf_idf > .009)

reviews_shortened |>
  group_by(review_type) |>
  summarise(count=n())
```

    # A tibble: 2 × 2
      review_type  count
      <chr>        <int>
    1 NEG         791011
    2 POS         442880

It’s ultimately not getting rid of *much* but any little step helps.

## Data Setup

Even after downsampling and removing stop words, I ran into issues
trying to pivot my very large dataframe wider.

Initially I sampled lines 1:400,000 of my data which worked well enough!
But when I started to train the classifier on it, ran into an issue
where even in all those rows, only POS marked reviews were appearing.

Instead, I did a random sampling method of 300,000 rows of data and
confirmed afterward that it contained both NEG and POS marked reviews.

setup_sample \<- sample_n(reviews_shortened, 800)

**Note that for rendered .md sampled slice is much smaller than in the
.qmd file because rendering kept timing out**

``` r
set.seed(14)
setup_sample <- reviews_shortened |>
  group_by(review_type) |>
  slice_sample(n = 10000)

setup_sample <-setup_sample |>
  rename(WC=total)
  
setup_sample
```

    # A tibble: 20,000 × 5
    # Groups:   review_type [2]
       review_id review_type word          WC tf_idf
           <dbl> <chr>       <chr>      <dbl>  <dbl>
     1 186153668 NEG         crash        179 0.0256
     2 129898197 NEG         stanley       34 0.135 
     3 143230050 NEG         complete      34 0.113 
     4 102010748 NEG         birkin        27 0.321 
     5   7748613 NEG         standalone   127 0.0573
     6  33441116 NEG         switched     164 0.0436
     7 193049277 NEG         review        47 0.0726
     8  15232499 NEG         force        114 0.0467
     9 183653186 NEG         refunded      10 0.557 
    10  10773061 NEG         bought        38 0.0936
    # ℹ 19,990 more rows

``` r
setup_sample |>
  group_by(review_type) |>
  summarise(count=n())
```

    # A tibble: 2 × 2
      review_type count
      <chr>       <int>
    1 NEG         10000
    2 POS         10000

Columns (words) with digits as the column name were creating an issue

``` r
setup_sample <- setup_sample |>
  mutate(word = str_replace_all(word, "(^\\d.*$)", "i_\\1"))
```

Pivot wider so that all the individual words appear as column names,
populated below by tf-idf values per review.

``` r
setup_sample <- setup_sample |>
  pivot_wider(names_from = word, values_from = tf_idf, values_fill = 0)

ncol(setup_sample)
```

    [1] 6794

Because of how the classifier works, any word that only appears once in
any of the reviews (has only one tf-idf value over 0) would be
discarded. If we train a classifier on the word “refund” and that word
only appears once in this data subset, it would never encounter the word
“refund” in the testing data to apply that classification weight to.
It’s wasted computing space and power on a word that it will never see
again.

To get rid of them in advance and ease the classification training
process, this chunk will discard any column (word) that only has one
value greater than 0 - any word that only appears a single time.

``` r
setup_sample <- setup_sample |>
  dplyr::select(!review_id)
```

The classifier needs this column to be factor type in order to run

``` r
setup_sample$review_type <- as.factor(setup_sample$review_type)
```

Remove any columns that have only one tf-idf value. It would indicate
that they only show up once in the dataframe. If it makes it into the
training split, it wouldn’t appear in the testing data and would be a
waste of training space. If it only appears in the testing data, it
won’t have been trained on it. A data point that only appears once is
not a useful one.

``` r
setup_sample <- setup_sample |>
  purrr::discard(function(x) sum(x != 0) == 1)

ncol(setup_sample)
```

    [1] 2768

``` r
head(setup_sample)
```

    # A tibble: 6 × 2,768
    # Groups:   review_type [1]
      review_type    WC  crash stanley complete birkin standalone review force
      <fct>       <dbl>  <dbl>   <dbl>    <dbl>  <dbl>      <dbl>  <dbl> <dbl>
    1 NEG           179 0.0256   0        0      0         0           0     0
    2 NEG            34 0        0.135    0      0         0           0     0
    3 NEG            34 0        0        0.113  0         0           0     0
    4 NEG            27 0        0        0      0.321     0           0     0
    5 NEG           127 0        0        0      0         0.0573      0     0
    6 NEG           164 0        0        0      0         0           0     0
    # ℹ 2,759 more variables: refunded <dbl>, bought <dbl>, pls <dbl>, set <dbl>,
    #   option <dbl>, poor <dbl>, badly <dbl>, patch <dbl>, cultures <dbl>,
    #   qtes <dbl>, real <dbl>, giving <dbl>, idea <dbl>, stand <dbl>, money <dbl>,
    #   role <dbl>, terrible <dbl>, unravel <dbl>, killed <dbl>, found <dbl>,
    #   tons <dbl>, inventory <dbl>, i_360 <dbl>, refund <dbl>, path <dbl>,
    #   battles <dbl>, fucking <dbl>, visuals <dbl>, keyboard <dbl>, steam <dbl>,
    #   keeping <dbl>, anymore <dbl>, aspects <dbl>, weird <dbl>, add <dbl>, …

One last step to help the classifier run and train properly… some small
elements like apostrophes were causing issues.

``` r
setup_sample <- janitor::clean_names(setup_sample)
```

## Building a Classifier

Split the data by review type into training and testing data, with 75%
of the data going to training.

``` r
# split into training and testing sets

inTrain <- createDataPartition(
  y = setup_sample$review_type,
  ## the outcome data are needed
  p = .75,
  ## The percentage of data in the
  ## training set
  list = FALSE
)
```

``` r
training <- setup_sample[ inTrain,]
testing  <- setup_sample[-inTrain,]

nrow(training)
```

    [1] 10227

``` r
ncol(training)
```

    [1] 2768

``` r
nrow(testing)
```

    [1] 3408

``` r
ncol(testing)
```

    [1] 2768

``` r
training |>
  group_by(review_type) |>
  summarise(count=n())
```

    # A tibble: 2 × 2
      review_type count
      <fct>       <int>
    1 NEG          5331
    2 POS          4896

The classifier will be trained on 41,000 reviews and tested on close to
14,000 to see how well it can, by words used in a review, predict if a
review itself is a positive or negative one.

## Training

I initially attempted to train on a larger subset of 400,000 samples,
but the training ran for 6+ hours without end and I decided to downsize
to 300,000. This training I ran overnight, so I don’t know how long it
actually took.

``` r
rf_model <- ranger(dependent.variable.name = "review_type", 
                      data = training, 
                      num.trees = 40, 
                      mtry = 3,
                      min.node.size = 3,
                      importance = "permutation", 
                      seed = 123,
                      respect.unordered.factors = "ignore")
```

## Testing

``` r
p.ra <- predict(rf_model, data=testing)
str(p.ra)
```

    List of 5
     $ predictions              : Factor w/ 2 levels "NEG","POS": 1 1 1 1 1 1 1 1 1 1 ...
     $ num.trees                : num 40
     $ num.independent.variables: num 2767
     $ num.samples              : int 3408
     $ treetype                 : chr "Classification"
     - attr(*, "class")= chr "ranger.prediction"

Here are the results for this classifier when I included word count as a
predicting variable. I had hope that, knowing the negative reviews are
longer on average, including it as a factor would help the classifier be
a little bit better at predicting the class of the review (POS or NEG).
However, it seems to have only helped marginally.

Despite all efforts to give my classifier the best chances at being
informatively trained and able to classify reviews, it still wants to
predict the Negative class for almost all reviews it’s fed to test on.

``` r
confusion_matrix <- rf_model$confusion.matrix
    print(confusion_matrix)
```

         predicted
    true   NEG  POS
      NEG 5326    5
      POS 4871   25

When run on the full available size (150,000 rows of each review class)
with all words and word count included as a predicting variable, the
classifier correctly predicted all 20,734 NEG reviews, but only
correctly predicted 24 POS reviews. It incorrectly classified 19,510 POS
reviews as NEG

When run without word count included as a variable, the classifier
correctly predicted 20,733 NEG class reviews and 15 POS class reviews.
It incorrectly classified 19,519 POS reviews as NEG, and incorrectly
classified 1 single NEG review as POS.
