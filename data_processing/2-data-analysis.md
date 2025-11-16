# 2-data-analysis


**Note**: Please run
[0-data-exploration](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/0-data-exploration.qmd)
and
[1-data-cleanup](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/1-data-cleanup.qmd)
first to create the version of the .csv file needed to start here.

``` r
reviews_df <- read_csv("../private/reviews_analyze.csv")
```

    Rows: 189873 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (4): language, review_type, review, merged_text
    dbl  (2): review_id, steam_id
    date (1): date

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
reviews_df <- reviews_df |>
  rename(tokens = merged_text)

reviews_df
```

    # A tibble: 189,873 × 7
       review_id language date       review_type review              steam_id tokens
           <dbl> <chr>    <date>     <chr>       <chr>                  <dbl> <chr> 
     1 205434212 english  2025-08-29 POS         The atmosphere and…   239200 the, …
     2 205407336 english  2025-08-28 NEG         Gameplay is lackin…   239200 gamep…
     3 205265837 english  2025-08-27 NEG         So, if you are hop…   239200 so, i…
     4 205236599 english  2025-08-26 POS         Eerie suspenseful …   239200 eerie…
     5 205099824 english  2025-08-24 NEG         just play the 1st …   239200 just,…
     6 205035565 english  2025-08-23 NEG         This game has good…   239200 this,…
     7 205004797 english  2025-08-23 POS         Absolute Cinema       239200 absol…
     8 204962454 english  2025-08-22 POS         Mostly a walking s…   239200 mostl…
     9 204909988 english  2025-08-22 NEG         I enjoyed the game…   239200 i, en…
    10 204877346 english  2025-08-21 NEG         Wish I could love …   239200 wish,…
    # ℹ 189,863 more rows

Copied over from the last file in the pipeline to keep track.

## List of Goals

- adjust the date column (DONE)
- clean up the text of non text items (DONE)
- Word tokens and word count per review (DONE)
- word types (DONE)
- TTR (DONE)

**Early Exploration**

- average review length (DONE)
- most common words (with and without stop words)
- most common words for pos and for neg
- some stats (correlation) - length and review type, review and rating
  type, maybe some others
- TF-IDF
- maybe let’s look at some emoji usage (STARTED)

**Later Goals**

- build a classifier

## Some Missed Cleanup

While I was working on word counts I discovered a number of unicode
blank spaces that were appearing as empty cells in the “review” column
and were being counted as 1 word reviews. I had to go through, find the
blank rows among the 1 word reviews, use charToRaw on the review ID
associated, and search online to find the unicode string to search for
and remove.

I went over the top to remove from both reviews and token columns, just
to be sure I was getting everything and removing the full row. It seemed
like when I didn’t, the rows weren’t being entirely eliminated. Overkill
is fine with me if the end result is what I’m after!

## Word Count

``` r
reviews_df <- reviews_df |>
  mutate(word_count = str_count(tokens, '\\,')+1)

reviews_df
```

    # A tibble: 189,302 × 8
       review_id language date       review_type review   steam_id tokens word_count
           <dbl> <chr>    <date>     <chr>       <chr>       <dbl> <chr>       <dbl>
     1 205434212 english  2025-08-29 POS         The atm…   239200 the, …         74
     2 205407336 english  2025-08-28 NEG         Gamepla…   239200 gamep…          9
     3 205265837 english  2025-08-27 NEG         So, if …   239200 so, i…        104
     4 205236599 english  2025-08-26 POS         Eerie s…   239200 eerie…        136
     5 205099824 english  2025-08-24 NEG         just pl…   239200 just,…          5
     6 205035565 english  2025-08-23 NEG         This ga…   239200 this,…        110
     7 205004797 english  2025-08-23 POS         Absolut…   239200 absol…          2
     8 204962454 english  2025-08-22 POS         Mostly …   239200 mostl…        220
     9 204909988 english  2025-08-22 NEG         I enjoy…   239200 i, en…         33
    10 204877346 english  2025-08-21 NEG         Wish I …   239200 wish,…         28
    # ℹ 189,292 more rows

### Review length

First, there are a number of reviews that are just a blank space that
were counted as having 1 word because of how I formulated the word count
column. I’m going to declare these columns NA. If there’s no text in the
review, there’s nothing there for me to analyze anyway.

``` r
summary(reviews_df$word_count)
```

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
       1.00    5.00   17.00   51.58   50.00 2182.00 

So the average review is 51 words, and the median 17 words.

Let’s look a little deeper.

``` r
reviews_df |>
  filter(word_count==1)
```

    # A tibble: 13,409 × 8
       review_id language date       review_type review  steam_id tokens  word_count
           <dbl> <chr>    <date>     <chr>       <chr>      <dbl> <chr>        <dbl>
     1 201605730 english  2025-07-07 POS         Peak      239200 peak             1
     2 200743403 english  2025-06-26 NEG         bad       239200 bad              1
     3 197815403 english  2025-05-21 POS         meh       239200 meh              1
     4 195500370 english  2025-04-24 POS         Best      239200 best             1
     5 194075241 english  2025-04-02 POS         Classic   239200 classic          1
     6 191186015 english  2025-02-26 POS         pig       239200 pig              1
     7 190284038 english  2025-02-15 POS         oink      239200 oink             1
     8 190005117 english  2025-02-12 NEG         idk       239200 idk              1
     9 186869243 english  2025-01-01 NEG         no        239200 no               1
    10 185722507 english  NA         POS         pig       239200 pig              1
    # ℹ 13,399 more rows

``` r
reviews_df |>
  filter(word_count>1700)
```

    # A tibble: 51 × 8
       review_id language date       review_type review   steam_id tokens word_count
           <dbl> <chr>    <date>     <chr>       <chr>       <dbl> <chr>       <dbl>
     1 169799070 english  2024-06-14 POS         im goin…   632470 im, g…       1980
     2 202889082 english  2025-07-26 POS         I think…  1703340 i, th…       1876
     3 198792433 english  2025-06-03 POS         the end…  1703340 the, …       1730
     4 196181590 english  2025-05-01 POS         the end…  1703340 the, …       1883
     5 195331550 english  2025-04-21 POS         The end…  1703340 the, …       1883
     6 194031541 english  2025-04-02 POS         The end…  1703340 the, …       1883
     7 192702260 english  2025-03-14 POS         THE END…  1703340 the, …       1883
     8 192121032 english  2025-03-06 POS         The end…  1703340 the, …       1882
     9 191174046 english  2025-02-25 POS         The End…  1703340 the, …       1880
    10 191032150 english  2025-02-24 POS         The end…  1703340 the, …       2182
    # ℹ 41 more rows

There’s quite a few one-word reviews! Some of them are pretty
reasonable… “Amazing”, “Spooky”, “Boring”, “Unplayable”. Short and
sweet, gets the point across well enough. Others are a little less
obvious. I saw a number of keyboard smash reviews, strings of numbers, I
saw one that was just a rabbit emoji. The last one might be a reference
I just don’t understand.

Looking instead at these longest reviews, I was really surprised! A huge
majority of these are exactly the same phrase repeated over and over,
and those repeated reviews are all for the same game, as well. I have to
do some investigation…

Checking my game info
[here](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/notes_and_info/0-gameinfo.csv),
I was able to confirm that these repeating reviews are all from “The
Stanley Parable: Ultra Deluxe”.

Looking further, I found some more information. [“The end is
never”](https://thestanleyparable.fandom.com/wiki/The_End_Is_Never...)
is a tagline for the game itself, and appears within the game multiple
times as a reference to the inescapable time loop the game’s protagonist
is stuck in. The phrase also apparently appears on the game’s loading
screens in a constant loop.

### Positive and Negative review length

First let’s get a look at what we’re working with. How many reviews of
each type are there?

``` r
ggplot(reviews_df, aes(x=review_type, fill=review_type )) + 
  geom_bar( width = 0.5) +
  scale_fill_brewer(palette = "Set2") +
  geom_text(stat = "count", aes(label = after_stat(count)), size = 3.5, vjust = 3, hjust = 0.5, position = "stack") +
  theme(legend.position="right")
```

![](2-data-analysis_files/figure-commonmark/plot-reviews-1.png)

There are a LOT more positive reviews than negative ones. This made me
curious, what about the total data. It’s possible positively reviewed
games have more reviews in general than neagtively reviewed games?

``` r
games <- read_csv("../notes_and_info/0-gameinfo.csv")
```

    Rows: 45 Columns: 8
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (4): game, rank, perc_positive, price
    dbl (2): year, steam_id
    num (2): total_reviews, eng_reviews

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
games |>
  group_by(rank) |>
  #merging all varieties of pos/neg together by removing the first word/rank
  mutate(rank = str_replace_all(rank, "\\w+ (\\w)", "\\1")) |>
  summarise(eng_reviews = sum(eng_reviews))
```

    # A tibble: 3 × 2
      rank     eng_reviews
      <chr>          <dbl>
    1 Mixed          10148
    2 Negative       22645
    3 Positive      699524

It looks like kind of yes!

My theory here just comes down to popularity. As a game releases and
players try it out and review it, it either begins to migrate through
the ranks of positive or negative. As new players find out about the
game, if it shows already that it’s being negatively reviewed, why would
they spend money to try it out themselves? I wouldn’t! On the other
hand, if players see that a game is getting positive reviews, they’ll
more likely try it and, in turn, also positively review it.

Some of the games in the higher ranks in this data are hugely beloved
games, while the negative ones are very likely largely forgotten to time
once they descended into the negative review rankings.

So, there it is. Positively ranked games have overall more reviews than
negatively ranked games, and there are more positive reviews than
negative ones in my data as a result. We’ll keep that in mind, moving
forward.

### Average Review Length

Now to what we came here to look into. Are positive reviews typically
longer, or negative reviews?

``` r
ggplot(reviews_df, aes(x=review_type, y = word_count)) + 
  stat_summary(fun = mean, geom = "bar", fill = "skyblue") +
  geom_text(aes(label = after_stat(sprintf("%.2f", y))), stat = "summary", fun = "mean", vjust = 3, hjust=0.5) +
  ylab("Avg Amount") +
  theme(legend.position="right")
```

![](2-data-analysis_files/figure-commonmark/plot-avg-len-1.png)

Now that’s really interesting to me! Despite having fewer reviews total,
and *all* of the longest reviews we looked at above being positive,
negative reviews are still quite a bit longer on average. People must
have a lot more to say when they dislike a game than when they like one!

I wonder if there’s a trend in shorter reviews that might be swinging
this average one way or the other.

``` r
reviews_df |>
  group_by(review_type) |>
  filter(word_count<6) |>
  summarise(word_count = sum(word_count))
```

    # A tibble: 2 × 2
      review_type word_count
      <chr>            <dbl>
    1 NEG              14477
    2 POS             112927

Looks like it’s pretty common for positive reviews to be shorter. This
could potentially drag down the overall average of positive reviews.
However… there are more short positive reviews, but there are also more
positive reviews in general, so maybe not.

38.7% of all negative reviews are 5 words or shorter, and 74.4% of all
positive reviews are. That’s quite a big chunk of reviews!

### Length Stats

correlation between review length and review type?

## Word Types & Count

``` r
remove_dupes <- function(x) {
  #split tokens to be iterated over
  words <- strsplit(x, " ")[[1]]
  #apply the unique() function per row
  unique_words <- unique(words)
  #re-collapse into rows
  paste(unique_words, collapse = " ")
}
```

``` r
reviews_df$types <- sapply(reviews_df$tokens, remove_dupes)
```

``` r
reviews_df <- reviews_df |>
  mutate(type_count = str_count(types, '\\,')+1)
```

## TTR

TTR is used to measure the variety of language used in a text. It’s not
exactly a measure of the complexity of a document, but can be considered
with other factors as an indicator of a writer’s language abilities.

TTR is measured by dividing the total number of word types (unique words
used) by total number of word tokens (all words used). A low score
(closer to 0) indicates a highly repetitive document, and a high score
(closer to 1) indicates a higher variety or words. A score of 1 would
mean that no words were repeated in the document.

TTR is very sensitive to document length. Too long, and documents taper
off. There are only so many content words to be used in a document,
eventually the highly repetitive function words will outnumber them.

[source](https://medium.com/@rajeswaridepala/empirical-laws-ttr-cc9f826d304d)

``` r
reviews_df <- reviews_df |>
  mutate(TTR = type_count/word_count)

reviews_df
```

    # A tibble: 189,302 × 11
       review_id language date       review_type review   steam_id tokens word_count
           <dbl> <chr>    <date>     <chr>       <chr>       <dbl> <chr>       <dbl>
     1 205434212 english  2025-08-29 POS         The atm…   239200 the, …         74
     2 205407336 english  2025-08-28 NEG         Gamepla…   239200 gamep…          9
     3 205265837 english  2025-08-27 NEG         So, if …   239200 so, i…        104
     4 205236599 english  2025-08-26 POS         Eerie s…   239200 eerie…        136
     5 205099824 english  2025-08-24 NEG         just pl…   239200 just,…          5
     6 205035565 english  2025-08-23 NEG         This ga…   239200 this,…        110
     7 205004797 english  2025-08-23 POS         Absolut…   239200 absol…          2
     8 204962454 english  2025-08-22 POS         Mostly …   239200 mostl…        220
     9 204909988 english  2025-08-22 NEG         I enjoy…   239200 i, en…         33
    10 204877346 english  2025-08-21 NEG         Wish I …   239200 wish,…         28
    # ℹ 189,292 more rows
    # ℹ 3 more variables: types <chr>, type_count <dbl>, TTR <dbl>

Here I’ll look at the range of TTR and see what I see.

## Emojis

A lot of work went into keeping those emojis, so I want to take a look
at them!

``` r
emoji_pattern <- "[\\\U0001F600-\\\U0001F64F\\\U0001F300-\\\U0001F5FF\\\U0001F900-\\\U0001F9FF\\\U00002600-\\\U000027BF]"
```

``` r
emojis <- reviews_df |>
  filter(grepl(emoji_pattern, types, perl=TRUE)) |>
  mutate(types = str_remove_all(types, "\\w|,|'|‘|’
                                 %|\\.|’|\\\036|－|❝|❞
                                 |‑"))

emojis
```

    # A tibble: 1,622 × 11
       review_id language date       review_type review   steam_id tokens word_count
           <dbl> <chr>    <date>     <chr>       <chr>       <dbl> <chr>       <dbl>
     1 201991629 english  2025-07-12 POS         "\U0001…   239200 "\U00…        284
     2 198777296 english  2025-06-03 POS         "While …   239200 "whil…        131
     3 190511090 english  2025-02-18 POS         "₍ᐢ･⚇･ᐢ…   239200 "₍, ᐢ…          7
     4 188485499 english  2025-01-22 POS         "⭐ Rati…   239200 "⭐, r…        269
     5 185896610 english  NA         POS         "You ha…   239200 "you,…        264
     6 179793197 english  2024-10-25 POS         "Amnesi…   239200 "amne…        248
     7 173009600 english  2024-07-22 POS         "A mixe…   239200 "a, m…        413
     8 169448060 english  2024-06-10 POS         "\U0001…   239200 "\U00…        158
     9 168612935 english  2024-05-30 POS         "\U0001…   239200 "\U00…          3
    10 166058531 english  2024-04-26 POS         "The ga…   239200 "the,…        348
    # ℹ 1,612 more rows
    # ℹ 3 more variables: types <chr>, type_count <dbl>, TTR <dbl>

Only 1,622 of our reviews contain some kind of emoji usage. Not as many
as I expected!

``` r
emoji_df <- emojis |>
  count(types, sort=TRUE)

emoji_df
```

    # A tibble: 1,274 × 2
       types                                                                       n
       <chr>                                                                   <int>
     1 "\U0001f44d"                                                               90
     2 "  \U0001f44d"                                                             24
     3 " ☐       ☑                                                           …    16
     4 " \U0001f44d"                                                              14
     5 " ☐       ☑                                                           …    12
     6 "    \U0001f44d"                                                           10
     7 " ☐      ☑                                                            …    10
     8 "♥"                                                                        10
     9 "\U0001f44d \U0001f44d"                                                    10
    10 "   \U0001f44d"                                                             9
    # ℹ 1,264 more rows

``` r
max_count <- emoji_df |>
  pull(n) |>
  max()


hchart(
  emoji_df, 
  "bar",
  hcaes(types, n), 
  name = "Count",
  # dataLabels = list(enabled = TRUE, style = list(fontWeight = "normal"))
  ) |>
  hc_xAxis(
    min = 0,
    max = 30,
    scrollbar = list(enabled = TRUE)
    ) |>
  hc_yAxis(
    max = max_count,
    title = list(
      text = "Count",
      align = "high"
      )
    ) |>
  hc_tooltip(
    headerFormat = "{point.key}",
    pointFormat = " {point.y}"
  ) |>
  hc_size(height = 700)
```

Cool
[source](https://jkunst.com/blog/posts/2021-01-10-using-emoji-with-highcharterhighcharts/)
for the emoji finding & charting code.

## Tf-IDF
