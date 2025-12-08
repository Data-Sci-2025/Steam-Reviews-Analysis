# data exploration


## Data Preview & First Looks

``` r
amnesia <- read_csv("../data/amnesia.csv")
```

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 11545 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 239200, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
amnesia
```

    # A tibble: 11,545 × 9
        ...1 `239200` ...3                 ...4  ...5     ...6   ...7  Reviews ...9 
       <dbl> <chr>    <chr>                <chr> <chr>    <chr>  <chr> <chr>   <chr>
     1    NA <NA>     <NA>                 <NA>  <NA>     <NA>   <NA>  <NA>    <NA> 
     2    NA <NA>     Reviews by languages <NA>  <NA>     <NA>   <NA>  Positi… Nega…
     3    NA <NA>     <NA>                 Total Positive Negat… Ratio The ga… The …
     4    NA <NA>     Total                11521 8008     3513   70%   The ga… The …
     5    NA <NA>     english              6324  4147     2177   66%   There … The …
     6    NA <NA>     schinese             170   134      36     79%   The ga… Ther…
     7    NA <NA>     russian              1846  1392     454    75%   The st… The …
     8    NA <NA>     koreana              104   66       38     63%   The sa… The …
     9    NA <NA>     german               329   245      84     74%   The gr… The …
    10    NA <NA>     japanese             18    12       6      67%   The ga… The …
    # ℹ 11,535 more rows

``` r
deadlyprem <- read_csv("../data/deadlyprem.csv")
```

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4594 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 247660, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
deadlyprem
```

    # A tibble: 4,594 × 9
        ...1 `247660` ...3                 ...4  ...5      ...6 ...7  Reviews  ...9 
       <dbl> <chr>    <chr>                <chr> <chr>    <dbl> <chr> <chr>    <chr>
     1    NA <NA>     <NA>                 <NA>  <NA>        NA <NA>  <NA>     <NA> 
     2    NA <NA>     Reviews by languages <NA>  <NA>        NA <NA>  Positiv… Nega…
     3    NA <NA>     <NA>                 Total Positive    NA Ratio The gam… The …
     4    NA <NA>     Total                4570  3026      1544 66%   The gam… The …
     5    NA <NA>     english              2942  1903      1039 65%   There a… The …
     6    NA <NA>     schinese             58    37          21 64%   The gam… Ther…
     7    NA <NA>     russian              579   455        124 79%   The sto… The …
     8    NA <NA>     koreana              11    4            7 36%   The san… The …
     9    NA <NA>     german               127   85          42 67%   The gra… The …
    10    NA <NA>     japanese             13    9            4 69%   The gam… The …
    # ℹ 4,584 more rows

First looks show some of the first areas I know I have to work around. I
called up two games just to confirm that they both look the same. Both
have the same almost metadata-like details at the top that will have to
be excluded, and both (and all the rest) of the files have that one odd
issue, with the column name being the game’s steam id (used to pull the
reviews in through the script)

``` r
print(colnames(deadlyprem))
```

    [1] "...1"    "247660"  "...3"    "...4"    "...5"    "...6"    "...7"   
    [8] "Reviews" "...9"   

## Early Tidying Steps

### colnames

``` r
deadlyprem <- deadlyprem |>
  rename(review_id = 1, language = 2, date = 3, review_type = 4, purchased = 5, score = 6, upvotes = 7, review = 8, engtxt = 9) |>
  filter(language == 'english')
```

Thinking about future column names! Not yet thinking deeply about what
columns I’ll ultimately drop, but I’m starting to weigh my options
(date, upvotes, engtxt)

Note: Think about how to rename cols using iteration for ease…. the
language column name comes from the game url id number used to pull the
reviews.

### str replace

``` r
deadlyprem |>
  distinct(review_type)
```

    # A tibble: 2 × 1
      review_type
      <chr>      
    1 ✅         
    2 ❌         

``` r
deadlyprem |>
  mutate(review_type =  str_replace_all(review_type, 
                c("✅" = "POS",
                  "❌" = "NEG")))
```

    # A tibble: 2,942 × 9
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <dbl> <chr>   <chr>  <chr> 
     1 205149678 english  2025.08… POS         ✅        0.524 2       "---{… "---{…
     2 205129562 english  2025.08… NEG         ✅        0.5   0       "I ca… "I ca…
     3 204814161 english  2025.08… NEG         ✅        0.524 1       "It s… "It s…
     4 204749451 english  2025.08… POS         ✅        0.5   1       "I wi… "I wi…
     5 204667995 english  2025.08… NEG         ✅        0.483 0       "I li… "I li…
     6 204591205 english  2025.08… POS         ✅        0.5   0       "I we… "I we…
     7 204510254 english  2025.08… POS         ✅        0.5   0       "I AM… "I AM…
     8 204388121 english  2025.08… POS         ✅        0.5   0       "Damn… "Damn…
     9 204256069 english  2025.08… POS         ✅        0.5   0       "This… "This…
    10 204246481 english  2025.08… NEG         ✅        0.550 5       "Prob… "Prob…
    # ℹ 2,932 more rows

For now, going with POS and NEG for easy visual differences. I found
“positive” and “negative” too messy and similar looking at a glance, I
think POS and NEG are easier to quickly tell apart.

## Game Info DFs

I go into some more detail about these in my
[projnotes](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/notes_and_info/projnotes.md)
file there. All this information comes directly from the Steam app. The
app only shows English reviews as a separate metric from all reviews at
a certain number of reviews. In those cases, I had to check the total
number of English reviews from the csv files individually.

``` r
overpos <- tibble(game = c("Outer wilds", "Stardew Valley", "Hades", "Chants of Sennaar", "The Case of the Golden Idol"), rank = "Overwhelmingly Positive", perc_positive = c("95%", "98%", "98%", "98%", "98%"), year = c("2020", "2016", "2020", "2023", "2022"), total_reviews = c("75,534", "797,573", "264,668", "22,874", "8,107"), eng_reviews = c("45,703", "363,715", "133,152", "10,464", "5,778"), price = c("$25", "$15", "$25", "$20", "$18"),  steam_id = c("753640", "413150", "1145360", "1931770", "1677770"))
overpos
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Outer wilds Over… 95%           2020  75,534        45,703      $25   753640  
    2 Stardew Va… Over… 98%           2016  797,573       363,715     $15   413150  
    3 Hades       Over… 98%           2020  264,668       133,152     $25   1145360 
    4 Chants of … Over… 98%           2023  22,874        10,464      $20   1931770 
    5 The Case o… Over… 98%           2022  8,107         5,778       $18   1677770 

``` r
verypos <- tibble(game = c("Disco Elysium", "Superliminal", "The Stanley Parable: Ultra Deluxe", "Dead Space 2", "Night in the Woods"), rank = "Very Positive", perc_positive = c("92%", "94%", "94%", "94%", "94%"), year = c("2019", "2020", "2022", "2011", "2017"), total_reviews = c("105,914", "25,549", "30,300", "22,870", "16,570"), eng_reviews = c("52,174", "13,202", "22,192","10,723", "12,697"), price = c("$40", "$20", "$25", "$20", "$20"), steam_id = c("632470", "1049410", "1703340", "47780", "481510"))
verypos
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Disco Elys… Very… 92%           2019  105,914       52,174      $40   632470  
    2 Superlimin… Very… 94%           2020  25,549        13,202      $20   1049410 
    3 The Stanle… Very… 94%           2022  30,300        22,192      $25   1703340 
    4 Dead Space… Very… 94%           2011  22,870        10,723      $20   47780   
    5 Night in t… Very… 94%           2017  16,570        12,697      $20   481510  

``` r
pos <- tibble(game = c("Powerful Courses", "Bike Ride 3D", "Tomorrow is my Birthday", "Hyper Space", "Ascending Inferno"), rank = "Positive", perc_positive = c("100%", "100%", "100%", "100%", "100%"), year = c("2023", "2024", "2024", "2024", "2024"), total_reviews = c("49", "49", "48", "33", "48"), eng_reviews = c("37", "43", "42","15", "53"), price = c("$15", "$4", "$3", "$5", "$15"), steam_id = c("2092330", "2607330", "2940020", "2444290", "2442820"))
pos
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Powerful C… Posi… 100%          2023  49            37          $15   2092330 
    2 Bike Ride … Posi… 100%          2024  49            43          $4    2607330 
    3 Tomorrow i… Posi… 100%          2024  48            42          $3    2940020 
    4 Hyper Space Posi… 100%          2024  33            15          $5    2444290 
    5 Ascending … Posi… 100%          2024  48            53          $15   2442820 

``` r
mostlypos <- tibble(game = c("Ranch Simulator", "Resident Evil 6", "Dark Pictures Anthology: Little Hope", "Blair Witch", "Final Fantasy VIII Remastered"), rank = "Mostly Positive", perc_positive = c("80%", "79%", "73%", "78%", "73%"), year = c("2023", "2013", "2020", "2019", "2019"), total_reviews = c("32,194", "41,151", "6,068", "5,898", "4,174"), eng_reviews = c("9,373", "12,266", "2,713","2,761", "2,421"), price = c("$25", "$20", "$20", "$30", "$20"), steam_id = c("1119730", "221040", "1194630", "1092660", "1026680"))
mostlypos
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Ranch Simu… Most… 80%           2023  32,194        9,373       $25   1119730 
    2 Resident E… Most… 79%           2013  41,151        12,266      $20   221040  
    3 Dark Pictu… Most… 73%           2020  6,068         2,713       $20   1194630 
    4 Blair Witch Most… 78%           2019  5,898         2,761       $30   1092660 
    5 Final Fant… Most… 73%           2019  4,174         2,421       $20   1026680 

``` r
mixed <- tibble(game = c("Deadly Premonition: The Director's Cut", "Silent Hill Homecoming", "Redfall", "Amneisa: A Machine for Pigs", "Vampire: The Masquerade - Coteries of New York"), rank = "Mixed", perc_positive = c("66%", "47%", "41%", "68%", "69%"), year = c("2013", "2008", "2023", "2013", "2019"), total_reviews = c("3,072", "2,379", "2,380", "6,807", "2,113"), eng_reviews = c("1,976", "1,099", "1,629","4,178", "1,266"), price = c("$25", "$40", "$40", "$20", "$20"), steam_id = c("247660", "19000", "1294810", "239200", "1096410"))
mixed
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Deadly Pre… Mixed 66%           2013  3,072         1,976       $25   247660  
    2 Silent Hil… Mixed 47%           2008  2,379         1,099       $40   19000   
    3 Redfall     Mixed 41%           2023  2,380         1,629       $40   1294810 
    4 Amneisa: A… Mixed 68%           2013  6,807         4,178       $20   239200  
    5 Vampire: T… Mixed 69%           2019  2,113         1,266       $20   1096410 

``` r
mostlyneg <- tibble(game = c("Roller Coaster Tycoon World", "Urban Empire", "Postal 3", "Towns", "Edge of Space"), rank = "Mostly Negative", perc_positive = c("25%", "34%", "39%", "27%", "32%"), year = c("2016", "2017", "2011", "2012", "2015"), total_reviews = c("3,179", "1,902", "4,328", "2,381", "1,945"), eng_reviews = c("1,944", "892", "1,984","2,058", "2,428"), price = c("$15", "$30", "$6", "$15", "$15"), steam_id = c("282560", "352550", "10220", "221020", "238240"))
mostlyneg
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Roller Coa… Most… 25%           2016  3,179         1,944       $15   282560  
    2 Urban Empi… Most… 34%           2017  1,902         892         $30   352550  
    3 Postal 3    Most… 39%           2011  4,328         1,984       $6    10220   
    4 Towns       Most… 27%           2012  2,381         2,058       $15   221020  
    5 Edge of Sp… Most… 32%           2015  1,945         2,428       $15   238240  

``` r
neg <- tibble(game = c("Bridge!", "Damned Nation Reborn", "Hazen: The Dark Whispers", "At Home", "Girl with a big SWORD"), rank = "Negative", perc_positive = c("12%", "10%", "20%", "18%", "16%"), year = c("2011", "2015", "2010", "2019", "2018"), total_reviews = c("47", "83", "48", "44", "43"), eng_reviews = c("31", "48", "41","26", "35"), price = c("$5", "$13", "$9", "$5", "$2"), steam_id = c("300160", "345710", "46730", "1048640", "946600"))
neg
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Bridge!     Nega… 12%           2011  47            31          $5    300160  
    2 Damned Nat… Nega… 10%           2015  83            48          $13   345710  
    3 Hazen: The… Nega… 20%           2010  48            41          $9    46730   
    4 At Home     Nega… 18%           2019  44            26          $5    1048640 
    5 Girl with … Nega… 16%           2018  43            35          $2    946600  

``` r
veryneg <- tibble(game = c("Lords of the Black Sun", "Skyscraper Simulator", "Citadels", "Construction Machines 2014", "Bhop Pro"), rank = "Very Negative", perc_positive = c("18%", "14%", "13%", "7%", "17%"), year = c("2014", "2010", "2013", "2014", "2020"), total_reviews = c("328", "266", "490", "239", "244"), eng_reviews = c("300", "479", "276","156", "69"), price = c("$25", "$3", "$15", "$7", "$10"), steam_id = c("246940", "252910", "238870", "252050", "1308950"))
veryneg
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Lords of t… Very… 18%           2014  328           300         $25   246940  
    2 Skyscraper… Very… 14%           2010  266           479         $3    252910  
    3 Citadels    Very… 13%           2013  490           276         $15   238870  
    4 Constructi… Very… 7%            2014  239           156         $7    252050  
    5 Bhop Pro    Very… 17%           2020  244           69          $10   1308950 

``` r
overneg <- tibble(game = c("Kinetic Void", "Spacebase DF9", "FlatOut 3 Chaos & Destruction", "Command & Conquer 4 Tiberian Twilight", "Superpower 3"), rank = "Overwhelmingly Negative", perc_positive = c("17%", "17%", "17%", "16%", "10%"), year = c("2014", "2014", "2011", "2010", "2022"), total_reviews = c("1,553", "4,313", "4,079", "4,711", "4,713"), eng_reviews = c("1,346", "3,675", "1,606","2,625", "2,626"), price = c("$10", "$10", "$10", "$20", "$30"), steam_id = c("227160", "246090", "201510", "47700", "1563130"))
overneg
```

    # A tibble: 5 × 8
      game        rank  perc_positive year  total_reviews eng_reviews price steam_id
      <chr>       <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
    1 Kinetic Vo… Over… 17%           2014  1,553         1,346       $10   227160  
    2 Spacebase … Over… 17%           2014  4,313         3,675       $10   246090  
    3 FlatOut 3 … Over… 17%           2011  4,079         1,606       $10   201510  
    4 Command & … Over… 16%           2010  4,711         2,625       $20   47700   
    5 Superpower… Over… 10%           2022  4,713         2,626       $30   1563130 

``` r
allgames <- bind_rows(overpos, verypos, pos, mostlypos, mixed, mostlyneg, neg, veryneg, overneg)
allgames
```

    # A tibble: 45 × 8
       game       rank  perc_positive year  total_reviews eng_reviews price steam_id
       <chr>      <chr> <chr>         <chr> <chr>         <chr>       <chr> <chr>   
     1 Outer wil… Over… 95%           2020  75,534        45,703      $25   753640  
     2 Stardew V… Over… 98%           2016  797,573       363,715     $15   413150  
     3 Hades      Over… 98%           2020  264,668       133,152     $25   1145360 
     4 Chants of… Over… 98%           2023  22,874        10,464      $20   1931770 
     5 The Case … Over… 98%           2022  8,107         5,778       $18   1677770 
     6 Disco Ely… Very… 92%           2019  105,914       52,174      $40   632470  
     7 Superlimi… Very… 94%           2020  25,549        13,202      $20   1049410 
     8 The Stanl… Very… 94%           2022  30,300        22,192      $25   1703340 
     9 Dead Spac… Very… 94%           2011  22,870        10,723      $20   47780   
    10 Night in … Very… 94%           2017  16,570        12,697      $20   481510  
    # ℹ 35 more rows

``` r
#write_csv(allgames, file="0-gameinfo.csv")
```

### Some quick stats

45 total games, how many reviews will I be looking at?

``` r
allgames <- allgames |>
  mutate(total_reviews = (str_replace_all(total_reviews, ",", ""))) |>
  mutate(total_reviews = as.numeric(total_reviews))
  
sum(allgames$total_reviews)
```

    [1] 1511358

``` r
allgames <- allgames |>
  mutate(eng_reviews = (str_replace_all(eng_reviews, ",", ""))) |>
  mutate(eng_reviews = eng_reviews |>
           na_if("NA")) |>
  mutate(eng_reviews = as.numeric(eng_reviews))
  
sum(allgames$eng_reviews, na.rm=TRUE)
```

    [1] 732317

I only plan to look at english reviews, but it took some work to
transform both columns into something sum-able.

## Getting the Data read in

Getting the files read in starts with the list of all the file names in
the data file:

``` r
filenames <- list.files("../data", pattern="*\\.csv$", full.names=TRUE)
filenames
```

     [1] "../data/amnesia.csv"      "../data/athome.csv"      
     [3] "../data/bhop.csv"         "../data/bigsword.csv"    
     [5] "../data/bikeride.csv"     "../data/birthday.csv"    
     [7] "../data/blacksun.csv"     "../data/blairwitch.csv"  
     [9] "../data/bridge.csv"       "../data/citadels.csv"    
    [11] "../data/coaster.csv"      "../data/command.csv"     
    [13] "../data/construction.csv" "../data/courses.csv"     
    [15] "../data/deadlyprem.csv"   "../data/deadspace.csv"   
    [17] "../data/disco.csv"        "../data/edgespace.csv"   
    [19] "../data/FFVIII.csv"       "../data/flatout.csv"     
    [21] "../data/goldenidol.csv"   "../data/hades.csv"       
    [23] "../data/hazen.csv"        "../data/homecoming.csv"  
    [25] "../data/hyperspace.csv"   "../data/inferno.csv"     
    [27] "../data/kinetic.csv"      "../data/littlehope.csv"  
    [29] "../data/nightwoods.csv"   "../data/outer-wilds.csv" 
    [31] "../data/postal.csv"       "../data/ranchsim.csv"    
    [33] "../data/reborn.csv"       "../data/redfall.csv"     
    [35] "../data/residentevil.csv" "../data/sennaar.csv"     
    [37] "../data/skyscraper.csv"   "../data/spacebase.csv"   
    [39] "../data/stanley.csv"      "../data/stardew.csv"     
    [41] "../data/superliminal.csv" "../data/superpower.csv"  
    [43] "../data/towns.csv"        "../data/urbanemp.csv"    
    [45] "../data/vampires.csv"    

## Building Function and Dataframe

Because the base .csv files all look the same, but are a bit odd, there
are some quirks to deal with in the function.

1.  because column 2 is the game’s unique steam ID it prevents them
    being read in all together, so we have to flatten them into one
    string with str_flatten

2.  renaming those columns as decided earlier

3.  because the .csv files also have some meta data already seen above,
    the review_id, score, and upvotes cols were causing issues. The
    early rows are <chr> and the later ones are <int> so we have to
    force it as an int until we ditch that header info altogether.

``` r
build_df <- function(x){
  gamelines <- read_lines(x)
  flatlines <- str_flatten(gamelines, "\n")
  games_df <- read_csv(flatlines)
  steam_id <- games_df |> select(2) |> colnames()
  games_df <- games_df |> mutate(steam_id = steam_id)
  games_df <- games_df |>
    rename(review_id = 1, language = 2, date = 3, review_type = 4, purchased = 5, score = 6, upvotes = 7, review = 8, engtxt = 9) |>
    filter(language == 'english') |>
    mutate(review_id = as.integer(review_id)) |> 
    mutate(score = as.integer(score)) |>
    mutate(upvotes = as.integer(upvotes))
}
```

Using the function, build up the dataframe, map the 45 different steam
ID numbers onto 45 different dataframes, and then bind them all
together!

``` r
games_df <- map((filenames), build_df)|> 
  bind_rows()
```

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 11545 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 239200, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    Rows: 77 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 1048640, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 268 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 1308950, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 71 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 946600, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 73 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 2607330, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 73 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 2940020, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 417 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 246940, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 3824 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1092660, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 77 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 300160, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 514 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 238870, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 5836 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 282560, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4735 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 47700, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 337 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 252050, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 74 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 2092330, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4594 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 247660, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 26686 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 47780, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 27622 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 632470, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 2745 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 238240, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4877 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1026680, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4103 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 201510, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 9116 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 1677770, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 32524 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1145360, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    Rows: 75 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 46730, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 3613 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 19000, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 74 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 2444290, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 81 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 2442820, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    Rows: 1577 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (9): ...1, 227160, ...3, ...4, ...5, ...6, ...7, Reviews, ...9

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 9185 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 1194630, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 18926 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 481510, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 35024 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 753640, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 5671 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 10220, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 35024 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1119730, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    Rows: 107 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 345710, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4591 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1294810, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 32621 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 221040, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 14424 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 1931770, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    Rows: 503 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 252910, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 4337 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 246090, ...3, ...4, ...5, ...7, Reviews, ...9
    dbl (2): ...1, ...6

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 16224 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 1703340, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 37619 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 413150, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 30841 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1049410, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `score = as.integer(score)`.
    Caused by warning:
    ! NAs introduced by coercion

    New names:
    Rows: 1230 Columns: 9
    ── Column specification
    ──────────────────────────────────────────────────────── Delimiter: "," chr
    (9): ...1, 1563130, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:
    • `` -> `...1`
    • `` -> `...3`
    • `` -> `...4`
    • `` -> `...5`
    • `` -> `...6`
    • `` -> `...7`
    • `` -> `...9`

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 2830 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 221020, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 2481 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 352550, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
    New names:

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 2964 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 1096410, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
games_df
```

    # A tibble: 191,109 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <int> <chr>    <chr>    <chr>       <chr>     <int>   <int> <chr>  <chr> 
     1 205434212 english  2025.08… ✅          ❌            0       1 "The … "The …
     2 205407336 english  2025.08… ❌          ❌            0       0 "Game… "Game…
     3 205265837 english  2025.08… ❌          ✅            0       0 "So, … "So, …
     4 205236599 english  2025.08… ✅          ❌            0       0 "Eeri… "Eeri…
     5 205099824 english  2025.08… ❌          ❌            0       1 "just… "just…
     6 205035565 english  2025.08… ❌          ✅            0       0 "This… "This…
     7 205004797 english  2025.08… ✅          ✅            0       0 "Abso… "Abso…
     8 204962454 english  2025.08… ✅          ✅            0       0 "Most… "Most…
     9 204909988 english  2025.08… ❌          ❌            0       0 "I en… "I en…
    10 204877346 english  2025.08… ❌          ✅            0       0 "Wish… "Wish…
    # ℹ 191,099 more rows
    # ℹ 1 more variable: steam_id <chr>

Just to verify, here they are! 45 different steam ID numbers for 45
different games. Thanks Dan for all the help!!

``` r
games_df |>
  distinct(steam_id)
```

    # A tibble: 45 × 1
       steam_id
       <chr>   
     1 239200  
     2 1048640 
     3 1308950 
     4 946600  
     5 2607330 
     6 2940020 
     7 246940  
     8 1092660 
     9 300160  
    10 238870  
    # ℹ 35 more rows

## Write out reviews_csv

``` r
#write_csv(games_df, file="../private/reviews.csv")
```

## Session Info

``` r
sessionInfo()
```

    R version 4.5.1 (2025-06-13 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 10 x64 (build 19045)

    Matrix products: default
      LAPACK version 3.12.1

    locale:
    [1] LC_COLLATE=English_United States.utf8 
    [2] LC_CTYPE=English_United States.utf8   
    [3] LC_MONETARY=English_United States.utf8
    [4] LC_NUMERIC=C                          
    [5] LC_TIME=English_United States.utf8    

    time zone: America/New_York
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    other attached packages:
     [1] lubridate_1.9.4 forcats_1.0.1   stringr_1.5.2   dplyr_1.1.4    
     [5] purrr_1.1.0     readr_2.1.5     tidyr_1.3.1     tibble_3.3.0   
     [9] ggplot2_4.0.0   tidyverse_2.0.0

    loaded via a namespace (and not attached):
     [1] bit_4.6.0          gtable_0.3.6       jsonlite_2.0.0     crayon_1.5.3      
     [5] compiler_4.5.1     tidyselect_1.2.1   parallel_4.5.1     scales_1.4.0      
     [9] yaml_2.3.10        fastmap_1.2.0      R6_2.6.1           generics_0.1.4    
    [13] knitr_1.50         pillar_1.11.1      RColorBrewer_1.1-3 tzdb_0.5.0        
    [17] rlang_1.1.6        utf8_1.2.6         stringi_1.8.7      xfun_0.53         
    [21] S7_0.2.0           bit64_4.6.0-1      timechange_0.3.0   cli_3.6.5         
    [25] withr_3.0.2        magrittr_2.0.4     digest_0.6.37      grid_4.5.1        
    [29] vroom_1.6.6        rstudioapi_0.17.1  hms_1.1.3          lifecycle_1.0.4   
    [33] vctrs_0.6.5        evaluate_1.0.5     glue_1.8.0         farver_2.1.2      
    [37] rmarkdown_2.29     tools_4.5.1        pkgconfig_2.0.3    htmltools_0.5.8.1 
