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
deathloop <- read_csv("../data/deathloop.csv")
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

    Rows: 29626 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): 1252330, ...3, ...4, ...5, ...6, Reviews, ...9
    dbl (2): ...1, ...7

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
deathloop
```

    # A tibble: 29,626 × 9
        ...1 `1252330` ...3                 ...4  ...5     ...6   ...7 Reviews ...9 
       <dbl> <chr>     <chr>                <chr> <chr>    <chr> <dbl> <chr>   <chr>
     1    NA <NA>      <NA>                 <NA>  <NA>     <NA>     NA <NA>    <NA> 
     2    NA <NA>      Reviews by languages <NA>  <NA>     <NA>     NA Positi… Nega…
     3    NA <NA>      <NA>                 Total Positive Nega…    NA The ga… The …
     4    NA <NA>      Total                29602 22574    7028     NA The ga… The …
     5    NA <NA>      english              17351 13110    4241     NA There … The …
     6    NA <NA>      schinese             5281  4009     1272     NA The ga… Ther…
     7    NA <NA>      russian              1928  1535     393      NA The st… The …
     8    NA <NA>      koreana              455   307      148      NA The sa… The …
     9    NA <NA>      german               1029  782      247      NA The gr… The …
    10    NA <NA>      japanese             128   88       40       NA The ga… The …
    # ℹ 29,616 more rows

First looks show some of the first areas I know I have to work around. I
called up two games just to confirm that they both look the same. Both
have the same almost metadata-like details at the top that will have to
be excluded, and both (and all the rest) of the files have that one odd
issue, with the column name being the game’s steam id (used to pull the
reviews in through the script)

``` r
print(colnames(deathloop))
```

    [1] "...1"    "1252330" "...3"    "...4"    "...5"    "...6"    "...7"   
    [8] "Reviews" "...9"   

## Early Tidying Steps

### colnames

``` r
deathloop <- deathloop |>
  rename(review_id = ...1, language = 2, date = ...3, review_type = ...4, purchased = ...5, score = ...6, upvotes = ...7, review = Reviews, engtxt = ...9) |>
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
deathloop |>
  distinct(review_type)
```

    # A tibble: 2 × 1
      review_type
      <chr>      
    1 ✅         
    2 ❌         

``` r
deathloop |>
  mutate(review_type =  str_replace_all(review_type, 
                c("✅" = "POS",
                  "❌" = "NEG")))
```

    # A tibble: 17,351 × 9
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr>   <dbl> <chr>  <chr> 
     1 205094312 english  2025.08… POS         ❌        0.5         0 "7/10… "7/10…
     2 205000779 english  2025.08… POS         ✅        0.5         0 "It w… "It w…
     3 204904346 english  2025.08… POS         ❌        0.5         0 "this… "this…
     4 204842131 english  2025.08… POS         ✅        50.0%       0 "This… "This…
     5 204675894 english  2025.08… NEG         ❌        0.34…       2 "My g… "My g…
     6 204663628 english  2025.08… NEG         ✅        0.50…       5 "I re… "I re…
     7 204491003 english  2025.08… NEG         ❌        0.34…       2 "Publ… "Publ…
     8 204486791 english  2025.08… POS         ✅        0.39…       0 "这游戏设… "这游戏设…
     9 204483066 english  2025.08… NEG         ✅        0.34…       2 "The … "The …
    10 204479218 english  2025.08… POS         ✅        0.5         0 "well… "well…
    # ℹ 17,341 more rows

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

So this is where I’ve been noodling around trying to get all my data
read in. It’s still very much a work in progress, but here’s the parts
of what I’ve been putting together and working on here and there.

There are still some scratch-y notes in here because my reading in of
the steam ID number is still not working properly, but I’ll clean that
up later once it’s all in working condition.

``` r
filenames <- list.files("../data", pattern="*.csv", full.names=TRUE)
filenames
```

     [1] "../data/amnesia.csv"         "../data/athome.csv"         
     [3] "../data/bhop.csv"            "../data/bigsword.csv"       
     [5] "../data/bikeride.csv"        "../data/birthday.csv"       
     [7] "../data/blacksun.csv"        "../data/blairwitch.csv"     
     [9] "../data/bridge.csv"          "../data/citadels.csv"       
    [11] "../data/coaster.csv"         "../data/command.csv"        
    [13] "../data/construction.csv"    "../data/courses.csv"        
    [15] "../data/deadlyprem.csv"      "../data/deadspace.csv"      
    [17] "../data/deathloop.csv"       "../data/disco.csv"          
    [19] "../data/edgespace.csv"       "../data/FFVIII.csv"         
    [21] "../data/flatout.csv"         "../data/goldenidol.csv"     
    [23] "../data/hades.csv"           "../data/hazen.csv"          
    [25] "../data/homecoming.csv"      "../data/hyperspace.csv"     
    [27] "../data/inferno.csv"         "../data/kinetic.csv"        
    [29] "../data/littlehope.csv"      "../data/nightwoods.csv"     
    [31] "../data/outer-wilds.csv"     "../data/postal.csv"         
    [33] "../data/powerful-course.csv" "../data/ranchsim.csv"       
    [35] "../data/reborn.csv"          "../data/redfall.csv"        
    [37] "../data/residentevil.csv"    "../data/sennaar.csv"        
    [39] "../data/skyscraper.csv"      "../data/spacebase.csv"      
    [41] "../data/stanley.csv"         "../data/stardew.csv"        
    [43] "../data/superliminal.csv"    "../data/superpower.csv"     
    [45] "../data/towns.csv"           "../data/urbanemp.csv"       
    [47] "../data/vampires.csv"       

Here’s where I was experimenting with a for loop to go through each
file, add it into an empty list, and then I could bind that list
together, but that wasn’t working either.

``` r
datalist <- list()

for (file in filenames) {
  gamefile <- read.csv(file, header=FALSE)
  datalist <- gamefile
}
```

The loop is obviously working, but the dfs aren’t being added to the
list. Looking at the example below, it’s only going through and saving
the last dataset (Vampire Masquerade)

``` r
testing <- rbind(datalist)

testing
```

``` r
read.csv("../data/vampires.csv")
```

Here are the notes from 10/28 on a new process I’m going to give a shot!
We’ll see how it gets going.

look at chapter 26

pipe into map create anon function in map () to pull the column title
into its own column

x\<- listfiles

x\|\> map((x)) x\|\> steam ID col take steam id from col 2 \|\>
mutate(steam_id = IDNO))\|\> rename(review_lang = 2) \|\> bindrows()

    LETTERS[1:9]


    lines <- read_lines(file)
    steam_id <- str (parse to steam id) (lines)
    data <- read_csv(lines[26:length(lines)])

    (lines[26:length(lines)]) turn this into a single string before putting into read_csv


    |> mutate(steam_id = steam_id)

``` r
numtest <- amnesia |> 
  select(2) |>
  colnames()

numtest
```

    [1] "239200"

``` r
amnesia <- amnesia |>
  mutate(steam_id = numtest)

amnesia
```

    # A tibble: 11,545 × 10
        ...1 `239200` ...3            ...4  ...5  ...6  ...7  Reviews ...9  steam_id
       <dbl> <chr>    <chr>           <chr> <chr> <chr> <chr> <chr>   <chr> <chr>   
     1    NA <NA>     <NA>            <NA>  <NA>  <NA>  <NA>  <NA>    <NA>  239200  
     2    NA <NA>     Reviews by lan… <NA>  <NA>  <NA>  <NA>  Positi… Nega… 239200  
     3    NA <NA>     <NA>            Total Posi… Nega… Ratio The ga… The … 239200  
     4    NA <NA>     Total           11521 8008  3513  70%   The ga… The … 239200  
     5    NA <NA>     english         6324  4147  2177  66%   There … The … 239200  
     6    NA <NA>     schinese        170   134   36    79%   The ga… Ther… 239200  
     7    NA <NA>     russian         1846  1392  454   75%   The st… The … 239200  
     8    NA <NA>     koreana         104   66    38    63%   The sa… The … 239200  
     9    NA <NA>     german          329   245   84    74%   The gr… The … 239200  
    10    NA <NA>     japanese        18    12    6     67%   The ga… The … 239200  
    # ℹ 11,535 more rows

``` r
lines <- read_lines(filenames)
data <- read_csv(filenames, lines[25:length((lines))])


data
```

## Building function

``` r
build_df <- function(x){
  gamelines <- read_lines(x)
  flatlines <- str_flatten(gamelines, "\n")
  games_df <- read_csv(flatlines)
  steam_id <- games_df |> select(2) |> colnames()
  games_df <- games_df |> mutate(steam_id = steam_id)
  games_df <- games_df |>
    rename(review_id = 1, language = 2, date = 3, review_type = 4, purchased = 5, score = 6, upvotes = 7, review = 8, engtxt = 9) |>
  filter(language == 'english')
}
```

``` r
games_df <- build_df(filenames)
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

    Rows: 433463 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (8): 239200, ...3, ...4, ...5, ...6, ...7, Reviews, ...9
    dbl (1): ...1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
games_df
```

    # A tibble: 210,637 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr> <chr>   <chr>  <chr> 
     1 205434212 english  2025.08… ✅          ❌        0.51… 1       "The … "The …
     2 205407336 english  2025.08… ❌          ❌        0.5   0       "Game… "Game…
     3 205265837 english  2025.08… ❌          ✅        0.5   0       "So, … "So, …
     4 205236599 english  2025.08… ✅          ❌        0.5   0       "Eeri… "Eeri…
     5 205099824 english  2025.08… ❌          ❌        0.52… 1       "just… "just…
     6 205035565 english  2025.08… ❌          ✅        0.5   0       "This… "This…
     7 205004797 english  2025.08… ✅          ✅        0.47… 0       "Abso… "Abso…
     8 204962454 english  2025.08… ✅          ✅        0.5   0       "Most… "Most…
     9 204909988 english  2025.08… ❌          ❌        0.46… 0       "I en… "I en…
    10 204877346 english  2025.08… ❌          ✅        0.46… 0       "Wish… "Wish…
    # ℹ 210,627 more rows
    # ℹ 1 more variable: steam_id <chr>

``` r
#write_csv(games_df, file="../private/reviews.csv")
```

``` r
games_df |>
  distinct(steam_id)
```

    # A tibble: 1 × 1
      steam_id
      <chr>   
    1 239200  

``` r
dlooplines <- read_lines("../data/amnesia.csv")
flatlines <- str_flatten(dlooplines, "\n")
colid <- read_csv(flatlines)
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
steam_id <- colid |> select(2) |> colnames()
colid <- colid |> mutate(steam_id = steam_id)
colid <- colid |>
  rename(review_id = ...1, language = 2, date = ...3, review_type = ...4, purchased = ...5, score = ...6, upvotes = ...7, review = Reviews, engtxt = ...9) |>
  filter(language == 'english')

colid
```

    # A tibble: 6,324 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr> <chr>   <chr>  <chr> 
     1 205434212 english  2025.08… ✅          ❌        0.51… 1       "The … "The …
     2 205407336 english  2025.08… ❌          ❌        0.5   0       "Game… "Game…
     3 205265837 english  2025.08… ❌          ✅        0.5   0       "So, … "So, …
     4 205236599 english  2025.08… ✅          ❌        0.5   0       "Eeri… "Eeri…
     5 205099824 english  2025.08… ❌          ❌        0.52… 1       "just… "just…
     6 205035565 english  2025.08… ❌          ✅        0.5   0       "This… "This…
     7 205004797 english  2025.08… ✅          ✅        0.47… 0       "Abso… "Abso…
     8 204962454 english  2025.08… ✅          ✅        0.5   0       "Most… "Most…
     9 204909988 english  2025.08… ❌          ❌        0.46… 0       "I en… "I en…
    10 204877346 english  2025.08… ❌          ✅        0.46… 0       "Wish… "Wish…
    # ℹ 6,314 more rows
    # ℹ 1 more variable: steam_id <chr>

``` r
filenames |>
      map(\(x) read_csv(x, show_col_types=FALSE, col_names = c("A", "B", "C", "D", "E", "F", "G", "H", "I"))) |>
      map(\(x) 
        x |>
          mutate(steam_id = colnames(x)[2]) |>
          rename(review_lang = 2) 
      ) |>
 head(2)


#colid[-(1:24),] |>


#dlooplines2 <- dlooplines[24:length((dlooplines))]
#dloopstr <- str_flatten(dlooplines2, "\n")
#dloopdata <- read_csv(dloopstr)
```
