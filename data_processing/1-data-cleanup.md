# analysis pipeline


**Note**: reviews.csv is too large to upload to github right now, but
using the data which is uploaded and the “data exploration” file, if you
un-comment the write_csv chunk, you should be able to recreate it just
fine. You’ll just have to change the file path to read in for this one.

``` r
reviews_df <- read_csv("../private/reviews.csv")
```

    Rows: 210637 Columns: 10
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (7): language, date, review_type, purchased, score, review, engtxt
    dbl (3): review_id, upvotes, steam_id

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
reviews_df
```

    # A tibble: 210,637 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr>   <dbl> <chr>  <chr> 
     1 205434212 english  2025.08… ✅          ❌        0.51…       1 "The … "The …
     2 205407336 english  2025.08… ❌          ❌        0.5         0 "Game… "Game…
     3 205265837 english  2025.08… ❌          ✅        0.5         0 "So, … "So, …
     4 205236599 english  2025.08… ✅          ❌        0.5         0 "Eeri… "Eeri…
     5 205099824 english  2025.08… ❌          ❌        0.52…       1 "just… "just…
     6 205035565 english  2025.08… ❌          ✅        0.5         0 "This… "This…
     7 205004797 english  2025.08… ✅          ✅        0.47…       0 "Abso… "Abso…
     8 204962454 english  2025.08… ✅          ✅        0.5         0 "Most… "Most…
     9 204909988 english  2025.08… ❌          ❌        0.46…       0 "I en… "I en…
    10 204877346 english  2025.08… ❌          ✅        0.46…       0 "Wish… "Wish…
    # ℹ 210,627 more rows
    # ℹ 1 more variable: steam_id <dbl>

In this current format, the steam id column is not correct, but I can
confirm all the games are there. The first reviews are for amnesia and
the ones at the end are for the Vampire: The Masquerade game. Once I fix
the steam_id workflow I’ll use distinct() to confirm there are 45
different game IDs. I could use that as well, later, to split training
and testing data and to narrow down to x number of games per game (where
possible).

``` r
reviews_df <- reviews_df |>
    mutate(review_type =  str_replace_all(review_type, 
                c("✅" = "POS",
                  "❌" = "NEG")))
```

Fix those pesky emoji columns

``` r
reviews_df
```

    # A tibble: 210,637 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr>   <dbl> <chr>  <chr> 
     1 205434212 english  2025.08… POS         ❌        0.51…       1 "The … "The …
     2 205407336 english  2025.08… NEG         ❌        0.5         0 "Game… "Game…
     3 205265837 english  2025.08… NEG         ✅        0.5         0 "So, … "So, …
     4 205236599 english  2025.08… POS         ❌        0.5         0 "Eeri… "Eeri…
     5 205099824 english  2025.08… NEG         ❌        0.52…       1 "just… "just…
     6 205035565 english  2025.08… NEG         ✅        0.5         0 "This… "This…
     7 205004797 english  2025.08… POS         ✅        0.47…       0 "Abso… "Abso…
     8 204962454 english  2025.08… POS         ✅        0.5         0 "Most… "Most…
     9 204909988 english  2025.08… NEG         ❌        0.46…       0 "I en… "I en…
    10 204877346 english  2025.08… NEG         ✅        0.46…       0 "Wish… "Wish…
    # ℹ 210,627 more rows
    # ℹ 1 more variable: steam_id <dbl>

``` r
diffrows <- reviews_df[reviews_df$review != reviews_df$engtxt, ]
diffrows
```

    # A tibble: 1,223 × 10
       review_id language date     review_type purchased score upvotes review engtxt
           <dbl> <chr>    <chr>    <chr>       <chr>     <chr>   <dbl> <chr>  <chr> 
     1 204166566 english  2025.08… POS         ❌        0.46…       0 7/10   45848 
     2        NA <NA>     <NA>     <NA>        <NA>      <NA>       NA <NA>   <NA>  
     3 196362582 english  2025.05… NEG         ✅        0.5         0 3/10   45726 
     4 194326563 english  2025.04… POS         ❌        0.5         0 5/10   45787 
     5 184768635 english  2025.00… POS         ❌        0.5         0 9/10   45910 
     6 168865908 english  2024.06… POS         ✅        0.5         0 9/10   45910 
     7 162686854 english  2024.03… NEG         ✅        0.41…       0 1/5    45662 
     8 155991617 english  2024.00… NEG         ❌        0.49…       1 3/10   45726 
     9        NA <NA>     <NA>     <NA>        <NA>      <NA>       NA <NA>   <NA>  
    10        NA <NA>     <NA>     <NA>        <NA>      <NA>       NA <NA>   <NA>  
    # ℹ 1,213 more rows
    # ℹ 1 more variable: steam_id <dbl>

``` r
reviews_df |>
  filter(review_id==204166566)
```

    # A tibble: 1 × 10
      review_id language date      review_type purchased score upvotes review engtxt
          <dbl> <chr>    <chr>     <chr>       <chr>     <chr>   <dbl> <chr>  <chr> 
    1 204166566 english  2025.08.… POS         ❌        0.46…       0 7/10   45848 
    # ℹ 1 more variable: steam_id <dbl>

the engtxt column is not actually different from the review column, the
only difference being conversions of any reviews that just say “xx%”
(100% being converted to 1) or anything with x/10 being also converted
to a string of numbers.

Now for column decisions. I’ll keep review_id since it’s a unique value
for each different review.

I don’t know that the date column is extremely important. I might want
to look at how quickly games get reviews (are they concentrated at
release and fall off, or is it relatively steady? any spikes?) but I
will probably edit the column to a better format. Ditch the time, look
into MM/DD/YY format instead, or whatever works best in R.

I don’t think the purchased col is extremely important, it only marks if
a review writer has purchased the game or not, which is not key to any
of my analysis.

score and upvotes is based on user response to a comment. Users can
upvote for helpfulness of a review, or mark it as funny. I can’t find
data on how exactly the review score is calculated, but I don’t think
these two factors will play a huge role in what I’m looking to do.

``` r
reviews_df <- reviews_df |>
  select(review_id:review_type, review, steam_id)
reviews_df
```

    # A tibble: 210,637 × 6
       review_id language date             review_type review               steam_id
           <dbl> <chr>    <chr>            <chr>       <chr>                   <dbl>
     1 205434212 english  2025.08.29 02:05 POS         "The atmosphere and…   239200
     2 205407336 english  2025.08.28 19:32 NEG         "Gameplay is lackin…   239200
     3 205265837 english  2025.08.27 00:41 NEG         "So, if you are hop…   239200
     4 205236599 english  2025.08.26 17:29 POS         "Eerie suspenseful …   239200
     5 205099824 english  2025.08.24 14:25 NEG         "just play the 1st …   239200
     6 205035565 english  2025.08.23 19:31 NEG         "This game has good…   239200
     7 205004797 english  2025.08.23 08:34 POS         "Absolute Cinema"      239200
     8 204962454 english  2025.08.22 21:33 POS         "Mostly a walking s…   239200
     9 204909988 english  2025.08.22 03:01 NEG         "I enjoyed the game…   239200
    10 204877346 english  2025.08.21 19:11 NEG         "Wish I could love …   239200
    # ℹ 210,627 more rows

``` r
reviews_df |>
  distinct(review_id)
```

    # A tibble: 207,975 × 1
       review_id
           <dbl>
     1 205434212
     2 205407336
     3 205265837
     4 205236599
     5 205099824
     6 205035565
     7 205004797
     8 204962454
     9 204909988
    10 204877346
    # ℹ 207,965 more rows

``` r
dupes <- duplicated(reviews_df$review_id)

dupes2 <- reviews_df[dupes, ]
dupes2[200:nrow(dupes2), ]
```

    # A tibble: 2,463 × 6
       review_id language date             review_type review               steam_id
           <dbl> <chr>    <chr>            <chr>       <chr>                   <dbl>
     1 190443226 english  2025.02.17 14:03 NEG         "Dodged it 15 years…   239200
     2 190427680 english  2025.02.17 07:56 NEG         "I bought this game…   239200
     3 190420231 english  2025.02.17 05:26 NEG         "I played this game…   239200
     4 190414049 english  2025.02.17 03:41 NEG         "This game is broke…   239200
     5 190395753 english  2025.02.16 23:31 NEG         "this one was garba…   239200
     6 190381054 english  2025.02.16 20:47 NEG         "This is a crime ag…   239200
     7 190301361 english  2025.02.16 01:49 NEG         "2 log-in screens a…   239200
     8 190167215 english  2025.02.14 18:08 NEG         "I don't even know …   239200
     9 190138069 english  2025.02.14 07:44 NEG         "This game no longe…   239200
    10 190067946 english  2025.02.13 08:57 NEG         "Ended up permanent…   239200
    # ℹ 2,453 more rows

``` r
reviews_df |>
  filter(review_id==148416513)
```

    # A tibble: 2 × 6
      review_id language date             review_type review                steam_id
          <dbl> <chr>    <chr>            <chr>       <chr>                    <dbl>
    1 148416513 english  2023.09.18 06:11 POS         "I've only played fo…   239200
    2 148416513 english  2023.09.18 06:11 POS         "I've only played fo…   239200

For some reason, some reviews appear to have been duplicated. I’m going
to revisit this once I fix my steam_id issue so I can pinpoint exactly
what is being duplicated here and why… It looks like the duplicated game
is command & conquer, but why….

I’ve been looking through some files and I can see that Command &
Conquer are duplicated, Powerful Courses are duplicated, that’s all I
can seem to find, though. I don’t know why these are duplicated, but
I’ll see what I can work out with some more searching and get rid of the
duplicates once I know for sure. Again once the steam_id problem is
fixed I’ll be able to verify that I have 45 different steam_ids and just
remove the duplicate value rows.

I verified that none of my files are identical to each other, double
checking by reading in any two that were similar in length and verifying
that the reviews don’t match. This is how I was able to verify that the
only two games with duplicates are C&C and Powerful Courses. Oddly, it
doesn’t seem like every single review for those games is duplicated,
only some of them. No idea why.

## List of Goals

- adjust the date column (DONE)
- clean up the text of non text items
- Word tokens and word count per review
- word types
- TTR

**Early Exploration**

- average review length
- most common words (with and without stop words)
- most common words for pos and for neg
- some stats (correlation) - length and review type, review and rating
  type, maybe some others
- TF-IDF

**Later Goals**

- build a classifier

## Date column

Date format as is: 2025.08.29 02:05

- remove timestamp
- reformat
- change column class

``` r
reviews_df <- reviews_df |>
  mutate(date = str_replace_all(date, "(\\d{4})\\.(\\d{2})\\.(\\d{2}).*", "\\1-\\2-\\3"))
reviews_df$date <- as.Date(reviews_df$date)  
  
reviews_df
```

    # A tibble: 210,637 × 6
       review_id language date       review_type review                     steam_id
           <dbl> <chr>    <date>     <chr>       <chr>                         <dbl>
     1 205434212 english  2025-08-29 POS         "The atmosphere and sound…   239200
     2 205407336 english  2025-08-28 NEG         "Gameplay is lacking, sto…   239200
     3 205265837 english  2025-08-27 NEG         "So, if you are hoping fo…   239200
     4 205236599 english  2025-08-26 POS         "Eerie suspenseful horror…   239200
     5 205099824 english  2025-08-24 NEG         "just play the 1st one"      239200
     6 205035565 english  2025-08-23 NEG         "This game has good lore …   239200
     7 205004797 english  2025-08-23 POS         "Absolute Cinema"            239200
     8 204962454 english  2025-08-22 POS         "Mostly a walking simulat…   239200
     9 204909988 english  2025-08-22 NEG         "I enjoyed the game as es…   239200
    10 204877346 english  2025-08-21 NEG         "Wish I could love this g…   239200
    # ℹ 210,627 more rows

## Text Cleaning

``` r
reviews_df |>
  select(review)
```

    # A tibble: 210,637 × 1
       review                                                                       
       <chr>                                                                        
     1 "The atmosphere and sound design are strong, but many core mechanics from Th…
     2 "Gameplay is lacking, story is nothing special. \nOverall meh."              
     3 "So, if you are hoping for an authentic Amnesia experience, this is going to…
     4 "Eerie suspenseful horror game which doesn't resemble other games of the ser…
     5 "just play the 1st one"                                                      
     6 "This game has good lore and scares but my main issue with this game is the …
     7 "Absolute Cinema"                                                            
     8 "Mostly a walking simulator, but there's a few puzzles. There's a few collec…
     9 "I enjoyed the game as essentially a creepy walking simulator with some ligh…
    10 "Wish I could love this game but it crashes at the end of the same level eve…
    # ℹ 210,627 more rows

``` r
reviews_df <- reviews_df |>
   mutate(review =  str_replace_all(review, 
                c("\n" = " ",
                  "\\(" = "",
                  "\\)" = "",
                  "(\\w)/(\\w)" = "\\1 \\2",
                  "(\\w) / (\\w)" = "\\1 \\2",
                  "\\[.*?\\]" = "",
                  "⠸|⢹|⠣|⣛|⣣|⣁|⣖|⣭|⠃|⢻|⣧|⣽" = "",
                  "⣷|⣟|⢶|⢣|⠄|⡿|⣮|⣻|⣦|⢛|⠿|⢭|⠼|⢀" = "",
                  "⡉|⣶|⠘|⢿|⡠|⢆|⣳|⡵|⡅|⠙|⣼|⡄|⡧|⢇" = "",
                  "⣴|⣌|⢽|⠻|⡛|⣤|⡝|⣾|⠚|⣀|⠏|⡻|⢘|⠎" = "",
                  "⣝|⣸|⣰|⡆|⣱|⣡|⢟|⣯|⡟|⢠|⠑|⠛|⢸|⢯" = "",
                  "⢎|⣕|⡮|⠝|⡾|⡯|⠕|⡹|⡂|⢈|⠟|⠢|⣎" = "",
                  "⡪|⡱|⢔|⡑|⢊|⢊|⠠|⠡|⠈|⡌|⠪|⣺|⡽" = "",
                  "⣗|⢉|⢄|⣐|⢂|⢐|⡔|⡀|⠁|⢑|⢨|⡁" = "",
                  "⢮|⣆|⠐|⣫|⢷|⢮|⢗|⢪|⣬|⢜|⢾|⣞" = "",
                  "⢗|⠥|⠨|⢪|⠂|⣬|⡗|⠧|⢞|⣪|⢱|⣬" = "",
                  "⡨|⡷|⢚|⠹|⡎|⢥|⠀|⡺|⢌|⡊|⡡|⠱|⠌" = "",
                  "⡲|⣹|⣑|⣨|⢳|⢡|⣜|⢵|⣔|⡸|⡭|⢃" = "",
                  "⢴|⢕|⡇|⢬|⢅|⣵|⠮|⢏|⡒|⡫|⡐|⡳" = "",
                  "⠯|⢢|⡩|⣃|⠔|⠜|⠇|⠅|⣇|⡰|⠰|⣅" = "",
                  "⡘|⡃|⢰|⡬|⡜|⢝|⢩|⡢|⡼|⣘|⠬|⠊|⢤" = "",
                  "⠫|⢼|⡕|⣲|⡏|⢺|⡈|⠤|⣥|⡥|⠶|⠽|⣥|⢦" = "",
                  "⠩|⡍|⠵|⣉|⣚|⠴|⠭|⣓|⡋|⠗|⣄|⣈|⠍" = "",
                  "⠺|⣢|⠷|⠋|⠾|⣠|⡴|⠆|⢖|⠞|⢲|⢁|⠉|⢋" = "",
                  "⢧|⣂|⢙|⠳|⣏|⡣|⡞|⠒|⡶|⣩|⠓|⣍|⠲|⡤" = "",
                  "⢫|⣙|⣊|⠖|⠦|⡙|⢓|⡦|⣒|⢒|⢍|⢒|⣋|⣋|⡖|⣿|⡚|⡓" = "",
                  "▄|░|┌|┼|▐|▀|▓|╝|╔|▌|╗|╚|═|║|▒|Ñ|│|∩|╓|╫" = "",
                  "╙|╡|╠|⊙|▅|┤|●|▂|▃|◥|█" = "",
                  
                  

                  "──+" = "─",
                  "__+" = "_",
                  
                  "  +" = " "
                  
                  )
                
                
                ))
```

I’m doing some exploring using str_view to dive in and look at what I
want to get rid of to begin some real analysis.

First - do I want to lowercase before tokenizing? I think yes but I have
to recheck old homeworks to be sure. Definitely want to lowercase before
looking at types and before doing tf-idf.

- how do you make a set of word types in R anyway i don’t know

Second - what to do with punctuation? I have to review previous work
also.

For the sake of processing text, I’m first removing any parenthesis
included. it could be interesting what gets put in a parenthetical
statement vs outside, but for pure analysis and vectorizing we don’t
want them.

There are some included kind of html(?) tags like \[b\] and \[/b\] of
\[h1\] etc. I want to get rid of those too.

Getting rid of /, sometimes it appears as word/word and others word /
word. Replacing it with a space and will later process to move extra
spaces down to one space - sometimes it appears in 7/10 or something.
I’m keeping those but I have to think about what to do with it. It also
depends on how the tokenizer handles it. If it tokenizes to \[7\] \[/\]
\[10\] I think that’s acceptable.

Questions - what do to with emojis?

Sometimes they are used stylistically to split up reviews: 📚 Story /
Narrative, ⌛ Length, 🔁 Replayability

Other times functionally (this is about difficulty): 🔲 Too easy ☑️
Casual-friendly 🔲 Balanced/Normal 🔲 Challenging 🔲 Brutal but fair 🔲
Pure chaos (PvP focus)

Sometimes in place of words: At least one review, start to finish,
contains one item only 💩 Another one looks like: This game is 💩 but i
still love it

Some reviews also have no words and will just say “7/10” and nothing
else, what about those?

What about abbreviations? I’ve seen af, wtf, omg, lmao, etc etc. - there
are a lot, i imagine just leave them as they are

There are also emoticons in the old style: :D, xD etc I even saw ° ͜ʖ
͡°, ﾉ◕ヮ◕ﾉ\*:･ﾟ✧

LOTS of formatting choices people make for visual purposes: =====, ☑ and
☐ (not as emojis),

Or what about this… There are a number of reviews using ⣿ (and all the
variations in the replace text chunk above) to create a kind of ASCII
art that can be safely removed, it is not speech information at all.

Think about \\.\*?\\ - there are some unicodes that show up I was having
trouble singling out so I have to figure that out. I think they’re
probably special characters used in some more elaborate emoticons/faces
that didn’t quite translate somewhere along the way.

links? Lots of people are linking to things in their reviews.

``` r
str_view(reviews_df$review, "/")
```

     [101] │ Amnesia: A Machine for Pigs is a well made and impressive game with a thorough thematic design and atmosphere. As a horror game it is rather spartan yet brilliant in that it let’s your brain conjure most of its horrors. Unfortunately it comes with a rather numbing effect that makes later parts of the game feel more formulaic and not as scary. https:</></>steamcommunity.com sharedfiles filedetails</>?id=3494853123
     [212] │  Amnesia: A Machine for Pigs Review Summary: ---------- This game was definitely a step back in the horror feel that The Dark Descent brought, and was more focused on story. There were definitely still some scary moments in this game, but it was story driven for sure. The game is very very short, and felt more like a DLC then a full game. The game took me around 6 hours to 100%, so causally playing will be even shorter. That being said, I still really enjoyed this game and the environment you play in. The choices are pretty straight forward and lead you to the exact result. I thought that the story was very interesting and was very tragic. Not as scary, but still worth the play. A firm 6 10 overall Storyline: ---------- This game follows Oswald Mandus, a wealthy industrialist who awakens with a fractured memory after a mysterious illness. He finds himself in his mansion, where strange and terrifying things are unfolding. As he explores the mansion and its underground factory, he uncovers disturbing truths about his involvement in a horrific experiment. He is talked to by someone named "The Engineer" telling him that his sons are trapped below in the machine. Mandus had been experimenting with a machine that could make huge amounts of pork to help combat the hungry. However, the machine became corrupted and also so did the people involved, turning them into things called Manpigs. The Engineerbetrays Mandus when having him turn the machine back on, and lets Manpigs loose onto the street taking and killing victims. As Mandus unravels his past, he realizes his own role in the monstrosities, including the fact that the machine is still running, powered by human suffering. He created the machine for human sacrifice to self mankind from itself. His evil and twisted soul is merged with the machine and creates "The Engineer". Like in the first game, an orb seems to have a tie in to the story. Mandus learn's everything of his past and decides that the machine must be stopped and destroy it for good. The Engineer tries to stop him to his best abilities, but fails, and Mandus succeeds in destroying the machine and them. It was a very interesting story, but also quite sad in the way that it is played out. I think the story obviously carries this game and is the best part. 8 10 Combat & Gameplay: ----------------------- The gameplay mechanics are very basic but in my opinion hold up very well and do not feel clunky at all. You carry a lantern and pick up items that can be important to the story that will be used from your inventory. This is exactly like the first game, and this feels even more refined then the first. 8 10 Collectibles: --------------- One of the achievements in the game is to get all of the notes, so you need to make sure as you progress you are grabbing them all. That is really the only collectible achievement in this game, so not too bad. Could have maybe used a little bit more, but that is okay. 6 10 I recommend following Division's guide, as he makes it very simple to find everything: http:</></>steamcommunity.com sharedfiles filedetails</>?id=2981456003 The Manpigs always looked a little funny walking around, made it easier to be less SCARED!!!!!! ; 
     [396] │ I've always heard really good things about Amnesia The Dark Descent but I almost never heard anything about Amnesia A Machine for Pigs and the few things that I did hear were not very positive. Well a long time ago I ended up buying both games and I played through The Dark Descent first. I really liked it and I thought it was a good horror game. Looking back on it after all these years I think I enjoyed the atmosphere, the mechanics and the gameplay more then the story itself. I stayed away from A Machine for Pigs for a while due to all the negative things I heard but after all these years it was about time I gave it a shot. So were all the bad things I heard correct? +The atmosphere is still incredibly good. The Dark Descent was great with its lighting and shadows and environment and the same applies here. Its definitely more advanced time-frame wise in The Dark Descent you had a lantern that you had to keep full of fuel and you could light candles along your way but in here? You have electricity and steam power! All the different lights and their flickering looks wonderful, the dark areas still look great as well and with the sounds combined into it all of it ends up creating a nice scary atmosphere. Aside from all the footsteps, lights flickering, doors closing and monsters making their own noises aside from all those the steam powered gigantic machine you end up exploring sounds great in its own way too. Hearing the pipes rattle and the occasional puff of steam being released is scary to hear when you're trying to not be spotted. More on that "not getting spotted" part later. +The game runs well, sounds incredibly well and plays without any issues. I never encounted a bug, a crash, had no technical difficulties and the controls were very easy and modern as expected. +The scale of things is incredible. This machine you're in definitely feels and looks huge. I love it. +The voice acting is really good. It definitely makes the dialogues much more fun to listen to and you get a lot more into the story thanks to it. Talking about the story... +The story is great. Its a little mixed at first but towards the end it really all comes together in a nice way. If you're gonna buy the game it should be for the story, for the voice acting and for the music. +</>- While the music is really good it only really gets good at the later parts of the game, towards the ending. Its sad that the music takes so long to get going but when it gets going oh it gets going alright. Its so fun to listen to just sucks that its in the last hour or so where it ends up really making an impact. +</>- While the voice acting is really good there are characters I dislike. The Kids. Granted making kids likeable or making them sound good in any game is a hard task on its own and if its not done really well it can stick out like a sore thumb. To me it wasn't that bad but they were definitely my least liked characters. Everyone else sounded really really well though. -I enjoyed The Dark Descent more for its gameplay and its mechanics rather then its story and in here the exact opposite is true. The gameplay, compared to The Dark Descent, has gone backwards. Less items for you to interact with, less puzzles and the few puzzles that you find are either super easy to complete or your character just writes everything in his diary so you can easily figure out what to do. No need to be careful with light anymore there's electrical lights all around the place and even your new, modern and improved lantern will never run out of fuel. Your character isn't afraid of the dark anyways so you can sit in a dark corner and wait for the monsters to go away for as long as you want without damaging your sanity like in The Dark Descent. Actually the entire sanity system has been thrown out the window as well as the inventory. An Amnesia game where I can't spam the TAB key to open my inventory when I get scared? Weird... -...No need to worry though aside from the atmosphere and loud noises there really isn't anything to scare you. Yes there are monsters but they're blind like a bat so sneaking around them is very easy. Even when you die the amount of progress you lose isn't much its like idk 5 seconds. Sometimes the monster won't even respawn after killing you so you can just go through the same area again but without being chased this time. That happened to me once and I would say its a bug but at the same time as soon as I made it out of there the paths I used just kinda dissapeared out of existance so my character might have just been having a psychotic episode or something. Being spotted is hard, being killed even harder. You might have 1-2 random jumpscares here and there and not much else. The things I heard about A Machine for Pigs were correct. The gameplay really is worse compared to The Dark Descent but after playing both I feel like people were exaggerating when they talked about how bad this game was. Its really not bad. The voice acting, the story and the music at the end definitely makes up for the lack of scary moments or the lack of interaction with random objects. I personally think that if it wasn't for the Amnesia title at the beginning of the game then people would see A Machine for Pigs as a much more succesfull game on its own. When it comes to if I'd recommend A Machine for Pigs over The Dark Descent... That depends on what you're looking for. Gameplay? Buy The Dark Descent. Story? Go for A Machine for Pigs.
     [418] │ A mixed bag of a game for sure, if you have played the first game, you will quickly realize this one isn't as well designed in its gameplay. But the music, art direction and story, truly were fantastic from the start, they weren't realized perfectly, but ultimately this is a game i would still recommend to anyone. This is one of those games you have to make peace with, it could've been better, but we still got something unique to experience. Just finish it once and see what i'm talking about. ---{ Graphics }--- ☐ You forget what reality is ☑ Beautiful ☐ Good ☐ Decent ☐ Bad ☐ Don‘t look too long at it ☐ MS-DOS ---{ Gameplay }--- ☐ Very good ☐ Good ☐ It's just gameplay ☑ Mehh ☐ Watch paint dry instead ☐ Just don't ---{ Audio }--- ☐ Eargasm ☑ Very good ☐ Good ☐ Not too bad ☐ Bad ☐ I'm now deaf ---{ Audience }--- ☐ Kids ☑ Teens ☑ Adults ☐ Grandma ---{ PC Requirements }--- ☐ Check if you can run paint ☐ Potato ☑ Decent ☐ Fast ☐ Rich boi ☐ Ask NASA if they have a spare computer ---{ Game Size }--- ☐ Floppy Disk ☐ Old Fashioned ☑ Workable ☐ Big ☐ Will eat 10% of your 1TB hard drive ☐ You will want an entire hard drive to hold it ☐ You will need to invest in a black hole to hold all the data ---{ Difficulty }--- ☐ Just press 'W' ☑ Easy ☐ Easy to learn Hard to master ☐ Significant brain usage ☐ Difficult ☐ Dark Souls ---{ Grind }--- ☑ Nothing to grind ☐ Only if u care about leaderboards ranks ☐ Isn't necessary to progress ☐ Average grind level ☐ Too much grind ☐ You'll need a second life for grinding ---{ Story }--- ☐ No Story ☐ Some lore ☐ Average ☐ Good ☑ Lovely ☐ It'll replace your life ---{ Game Time }--- ☐ Long enough for a cup of coffee ☐ Short ☑ Average ☐ Long ☐ To infinity and beyond ---{ Price }--- ☐ It's free! ☐ Worth the price ☑ If it's on sale ☐ If u have some spare money left ☐ Not recommended ☐ You could also just burn your money ---{ Bugs }--- ☐ Never heard of ☑ Minor bugs ☐ Can get annoying ☐ ARK: Survival Evolved ☐ The game itself is a big terrarium for bugs ---{ ? </> 10 }--- ☐ 1 ☐ 2 ☐ 3 ☐ 4 ☐ 5 ☐ 6 ☐ 7 ☑ 8 ☐ 9 ☐ 10
     [475] │ "He who makes a beast of himself from the of being a human". Dr. Samuel Johnson After I finally beat Amnesia: The Dark Descent TDD, it was finally time for me to play the next game in the series, Amnesia: A Machine For Pigs AMFP. This game at the time was not liked by the fans of the original game due to it not being like the first one. Frictional Games did not develop this game, as they were making SOMA then. Instead, The Chinese Room were the developers of this game, and it shows in some areas. Pros and Cons Pros A surprisingly good story The music and atmosphere are very well done Cons Does not work well, as a horror game The gameplay is a big downgrade from the previous entry, being mostly a walking sim Resource management has been removed The lantern does not do a good job of helping me see Puzzles don’t feel enjoyable AI is very bad Lacks the sense of fear that the first title had Very dark and hard to see in some areas Story Set in London 1899 on New Year's Eve, you play as Oswald Mandus who is a wealthy industrialist. He has recently returned from a disastrous expedition in Mexico where a tragedy struck. He was struck with a fever and had frequent dreams of this machine, he wakes up one night, months have passed, and he hears his two kids calling him to find them. https:</></>steamcommunity.com sharedfiles filedetails</>?id=3275704417 The story for AMFP is well thought out and I liked the narrative that the game had going for it. There are some small references to the previous game although more on that later that I did like very much. Although I do have one issue with the story and it’s just not really that scary and, sadly, this is the case. I feel that something is here that the developers had but I just did not have that sense of dread or fear while I was playing. Gameplay AMFP is a walking sim and that’s it that's the game has going for it, It does not work as a horror game. I say that the gameplay here is a huge downgrade to TDD and a lot of things that I loved about the first game, are gone from this one. You go from point A to B and pick up objects to progress through to the next area. The resource management system has also been removed. There are no tinderboxes, and your lantern does not run out of fuel either, it just removed one of the mechanics that made the first game a survival horror game. The insanity meter has also been removed as well, so you can look at the monsters without issue. Personally, I don’t mind that the sanity meter has been removed, you can make a horror game scary without it and it’s been done before many times. Then we get onto the puzzles the game has, or what the game calls puzzles because to me, they aren’t puzzles, they are not fun nor are they enjoyable in the slightest. You are just inserting objects into a machine or something and that’s it. The AI on the monsters is very bad, I can be right next to one and they won’t notice me and one of them just moved in a straight line at this one part of the game. Which I ran past, and it never noticed me whatsoever. There are very few encounters with the monsters in the game and this just leads to my next issue. AMFP is just not scary in the slightest, I just did not feel any sort of fear or dread like I did in TDD. I don’t feel that a lot is against me as there is nothing really to this game that says this is an Amnesia game. In fact, the only thing that says this is an Amnesia game is that the Orb from the previous game is mentioned. However, it’s only mentioned a few times throughout the game The game is also very dark, TDD was dark too but in AMFP I have a very hard time seeing things in the game. The lantern you get does not help you see in any way, and I just want the lantern back from TDD. Graphics and Performance The graphics for AMFP is well done, the atmosphere is really nice, and The Chinese Room nailed it in this regard. I tested the game on an NVIDIA GeForce GTX 1060 3GB, AMD Ryzen 5 1500X Quad-Core Processor 3.50, and 16 GB of RAM I experienced zero issues while I was playing, and the game ran fine. https:</></>steamcommunity.com sharedfiles filedetails</>?id=3275704663 Final Verdict In all honestly, if the name Amnesia, was not in this game's title, I may not have had this many negative things to say about this game. I think this game may have been good but with it carrying the name, it just feels like a disappointment. Amnesia: A Machine For Pigs does not work as a horror game and I think horror fans will be disappointed in this one, especially for those who just got done playing the TDD. Waiting for a sale or lucky find when the price is low is highly recommended. There are reviews at Life Needs a Little Sin that would make those clever fellows at the institute reel with envy
     [501] │ The gameplay wasn't the best but the story was amazing. Definitely worth a try. The atmosphere is great and the soundtrack is just wonderful ---{ Graphics }--- ☐ You forget what reality is ☐ Beautiful ☐ Good ☑ Decent ☐ Bad ☐ Don‘t look too long at it ☐ MS-DOS ---{ Gameplay }--- ☐ Very good ☐ Good ☐ It's just gameplay ☑ Mehh ☐ Watch paint dry instead ☐ Just don't ---{ Audio }--- ☑ Eargasm ☐ Very good ☐ Good ☐ Not too bad ☐ Bad ☐ I'm now deaf ---{ Audience }--- ☐ Kids ☐ Teens ☑ Adults ☐ Grandma ---{ PC Requirements }--- ☐ Check if you can run paint ☐ Potato ☑ Decent ☐ Fast ☐ Rich boi ☐ Ask NASA if they have a spare computer ---{ Game Size }--- ☐ Floppy Disk ☐ Old Fashioned ☑ Workable ☐ Big ☐ Will eat 10% of your 1TB hard drive ☐ You will want an entire hard drive to hold it ☐ You will need to invest in a black hole to hold all the data ---{ Difficulty }--- ☑ Just press 'W' ☐ Easy ☐ Easy to learn Hard to master ☐ Significant brain usage ☐ Difficult ☐ Dark Souls ---{ Grind }--- ☑ Nothing to grind ☐ Only if u care about leaderboards ranks ☐ Isn't necessary to progress ☐ Average grind level ☐ Too much grind ☐ You'll need a second life for grinding ---{ Story }--- ☐ No Story ☐ Some lore ☐ Average ☐ Good ☐ Lovely ☑ It'll replace your life ---{ Game Time }--- ☐ Long enough for a cup of coffee ☐ Short ☑ Average ☐ Long ☐ To infinity and beyond ---{ Price }--- ☐ It's free! ☑ Worth the price ☐ If it's on sale ☐ If u have some spare money left ☐ Not recommended ☐ You could also just burn your money ---{ Bugs }--- ☐ Never heard of ☑ Minor bugs ☐ Can get annoying ☐ ARK: Survival Evolved ☐ The game itself is a big terrarium for bugs ---{ ? </> 10 }--- ☐ 1 ☐ 2 ☐ 3 ☐ 4 ☐ 5 ☐ 6 ☐ 7 ☐ 8 ☑ 9 ☐ 10 ---{ Author }--- ☑ https:</></>vojtastruhar.github.io steam-review-template
     [587] │ "The industrial revolution and its consequences have been a disaster for the human race" - Ted Kaczynski Amnesia: A Machine for Pigs is very interesting entry to the Amnesia series, developed by The Chinese Room, AMFP has garnered a reputation within the series as a disappointing entry to the series. Truth be told, in comparison to The Dark Descent, sure it is a let down, but the hate this game has received is perhaps overblown. If you're familiar with its development AMFP originally started out as a mod before gaining approval from frictional games, with a staff consisting mostly of 2 people and were given little guidance on the series lore, along with content being cut due to time and budget constraints, so some of the faults of the game I can't blame too harshly. But in saying that their is some good that gets overlooked that needs to be highlighted. considering that it goes into a different direction one that doesn't have a supernatural element to the story is a commendable effort. The Good The story of AMFP centres around the turn of the century 1899, with rapid advancements in technology bringing out the worst in humanity and creating uncertainty for our future. You play as Oswald Mandus, after regaining consciousness, Oswald heeds to the beckoning of his children and a mysterious voice to unlock a dark a truth of his past. The narrative and themes of AMFP is explored decently. It's quite apparent that the focus of AMFP is to focus on its story which is interesting in concept but its execution is whole other can of worms. The ending of the game is so well presented that it almost makes you forget how convoluted and pretentious the rest of the story is. The soundtrack is pretty decent and feels well incorporated into its time period. The Bad AMFP relies too heavily on jump scares throughout the game, rather than just incorporating it into its atmosphere see The Ugly. Jump scares are fine if its done occasionally and or appropriately to help create an element of surprise or tension into the story to help make the audience more engaged with the narrative & environment. However, if overused which unfortunately AMFP does, then this looses its value and ends up becoming more shallow. After a while you keep expecting, and even predict, jump scares to happen and thus begin to loose their impact and disengage the audience jump scares become cheap when overused. The enemies in this game don't pose a huge threat as the previous Amnesia game did, as they can easily be out manoeuvred as long as you don't shine your lamp light on them for too long, the A.I. isn't all that intelligent either. Hiding is completely useless in the game, as their is no sanity metre along with enemies having a short focal range, you're better off just running everywhere which lessens the impact of the horror and fear of the game. There is little to no sense of urgency in the game, the lamp has no fuel to replenish, health replenishes automatically and as mentioned previously their is no sanity metre, which takes out a lot of the challenge and dread that TDD was able to accomplish. The story of AMFP is not executed very well aside from its ending, their is a plethora of plot holes and unexplained elements to its narrative that leads you to scratching your head in confusion you made a machine, then break the machine, get yourself amnesia to cure a fever, fix the machine, only to break the machine again shortly after, pretty underwhelming :</> . The diary entries in this game are told in riddles which only further confuses and clutters the narrative, while the audio logs are little more straight forward the way they're presented contrasts greatly with how the diary logs are written, which feels jarring. The Ugly The atmosphere of AMFP is good, for the most part. But can't help but figure that the reason why the atmosphere is serviceable is only because the game is way too dark. Because throughout the majority of the game is shrouded in darkness, this inhibits your visibility. So naturally your bound to be on edge, because you can't see whats in front of you most of the time. This comes off as a little superficial, for comparison TDD manages to invoke fear and an apprehensive atmosphere despite using different colours and lighting throughout some of its stages, creating a more unique yet horrifying experience. While the over reliance of darkness in this game doesn't make the environments all that memorable, especially when the vast majority of the game is spent in a large factory it all just sort of blends in with one another. The puzzles in the game are perhaps too simple and wouldn't be leaving a lasting impression after playing them, some of them are so obvious you wonder if they made this part a puzzle as an after thought. Ratings + Well fitting soundtrack + Monsters are decently designed + Very interesting ending +</>- Short game length 3 - 4 hours +</>- Good atmosphere though perhaps too dark +</>- Puzzles can be too simple +</>- Looses the supernatural element in the series for pseudoscience - Relies more on jump scares than atmosphere less scary - Enemies don't pose a huge threat can be easily dealt with - No sense of urgency hiding is pointless, no resources to manage, etc. - Dialogue can be inconsistent and pretentious at times often speaking in riddles - Story is very convoluted - Not as interactive or well designed as Amnesia TDD 5 10 - Just get it on sale
     [622] │ Here I will leave the cat, friends who pass by can pet it and give it a thumbs up and awards. {\u3000\u3000\u3000} {\u3000\u3000}／＞{\u3000\u3000}フ {\u3000\u3000\u3000} {\u3000\u3000}| {\u3000}_{\u3000} _ l {\u3000} {\u3000\u3000} {\u3000}／` ミ＿xノ {\u3000\u3000} {\u3000} </>{\u3000\u3000\u3000} {\u3000} | {\u3000\u3000\u3000} </>{\u3000} ヽ{\u3000\u3000} ﾉ {\u3000} {\u3000} {\u3000\u3000}|{\u3000}|{\u3000}| {\u3000}／￣|{\u3000\u3000} |{\u3000}|{\u3000}| {\u3000}| ￣ヽ＿_ヽ_ {\u3000}＼二つ
     [649] │ READ THE FULL REVIEW Steam has a word limit HERE: https:</></>gemsimov.com game-reviews f</>amnesia-a-machine-for-pigs-%7C-a-review -- Simple review details - I rank games on an out of 10 basis, granting up to 3 points in 3 categories, as well as a last, single point from my own self, depending on my experience with it. GAMEPLAY Amnesia: A Machine for Pigs AAMP is the sequel to Amnesia: The Dark Descent ATDD and as such it has some very massive boots to fill. Unfortunately, it does not succeed in filling said boots. A lot of the charm of ATDD is removed – there is no inventory system, there is no resource management and an infinite lightsource, most of the physics-based interactions with objects are no longer present and, overall, the game is very confined. It is, for all intents and purposes, a spicy walking simulator, with the Player being able to interact with objects that are key to solving some kind of puzzle and then being threatened, occasioanlly, by enemies don’t feel anywhere nearly as threatening or dangerous as those found in ATDD. On top of that, the game features sluggish movement, coupled with an odd inconsistency when it comes to the Player Character’s ability to move around – such as being able to jump on some objects and then being unable to jump on the same object. In addition to that, there are bugs to be encountered – such as falling out of bounds. Those would not have been too problematic if the rest of the game’s elements were not also simplified. Puzzles in AAMP were a lot easier and rudimentary than those found in ATDD, or so it felt. Ultimately, it can be classed as a downgrade in every regard, when it comes to its Gameplay. 2 3 PRESENTATION AAMP is reminiscent of its predecessor, but it fails in an important regard. This game is louder, and also relies on jump-scares. It also features a plentitude of moments during which the Player loses control, which are very inconsequential and make the decision of adding them very odd. While the game occasionally succeeds in delivering proper horror through well-designed instances, these moments are sparse. In summary, the game has a good visual appearance and some positive presentation elements but falls short in critical areas, leaving it somewhat lacking overall. 2 3 STORY This one also features an amnesiac who has had a lot to do with the troubles at hand, but it unfortunately fails to feature the complexity of its predecessor. For some reason the mystery that is present fails to be as captivating as ATDD, and the stakes at hand, although far higher on a macro level, lack this personal touch. The hook – which is the Player Character’s children – is not too intriguing, because it is quite clear that things are not going to turn out well for them. Overall it is unsatisfying and quite middling. 1 3 Legendary Point Does this game get the legendary point, so craved and wanted by all and none at the same time? Unfortunately, the answer is NO. While ATDD got the Legendary Point handily, AAMP does not manage to do that, due to the fact that it is more of a walking simulator than a game, due to the fact that it relies on jump-scares far more than ATDD and is brasher and louder than its predecessor, and due to the fact that the story is convoluted and fails to grab the Player as does the story in ATDD. 0 1 CONCLUSION 5 10. An average horror game. It does not do anything spectacular, aside from having the “Amnesia” name associated with it, and as such is not worth anyone’s time… Well, except for huge fans of Amnesia or the horror genre who just need to experience everything out there. In the bag of mediocrity I chuck it. It could have been so much better, but, alas, it simply was. -- READ THE FULL REVIEW Steam has a word limit HERE: https:</></>gemsimov.com game-reviews f</>amnesia-a-machine-for-pigs-%7C-a-review
     [675] │ I will leave the cat here, so that everybody who passes by can pet it and give it a thumbs up and awards {\u3000\u3000\u3000} {\u3000\u3000}／＞{\u3000\u3000}フ {\u3000\u3000\u3000} {\u3000\u3000}| {\u3000}_{\u3000} _ l {\u3000} {\u3000\u3000} {\u3000}／` ミ＿xノ {\u3000\u3000} {\u3000} </>{\u3000\u3000\u3000} {\u3000} | {\u3000\u3000\u3000} </>{\u3000} ヽ{\u3000\u3000} ﾉ {\u3000} {\u3000} {\u3000\u3000}|{\u3000}|{\u3000}| {\u3000}／￣|{\u3000\u3000} |{\u3000}|{\u3000}| {\u3000}| ￣ヽ＿_ヽ_ {\u3000}＼二つ
     [747] │ ﾉ◕ヮ◕ﾉ*:･ﾟ✧ https:</></>youtu.be FjXHpXFAv54
     [756] │ OK, let's get the important thing out of the way: This is NOTHING like Amnesia: Dark Descent. Where the first game was utterly terrifying due to being hunted in the darkness, this game is gruesome horror through suggestive storytelling and some nasty scenes of human</> animal butchery. This is more of a walking simulator through the horrific mind of a deranged lunatic, than the tight horror game of the original. Yes, there are sneak-em-up sections where you have to get through darkened sections past 'monsters' there being only two types I think, and there are lots of creepy dark areas like the original, but this one has more puzzle elements....Well, I say puzzle, it's more like "realise you have to do something X times, in X different places, and then do them. Sometimes you have to do them in order, sometimes you don't... But in the ordered ones, the game does a good job of restricting your paths so you don't get lost. In this, there are a few times you get turned around because one section looks identical to another, but the game instant-teleports you between places to disorient.... The story is very disturbing, and easily nightmare-inducing if you're prone to it. The level design is..... of its time. It feels early 2000's in the monster and level design. The music and sound effects are the best part - the music in the middle section of the game really sets your teeth on edge. The game is almost entirely in the dark, so you need a lamp. In most cases this is fine, but there are one or two areas e.g. the crypts where it's almost impossible to figure out where you're going, especially if you get 'caught' and get teleported to somewhere else... So, in a couple of areas, there's no shame in whacking up the gamma until you're out of them. The puzzles, on the whole, are very predictable and well telegraphed by the environmental storytelling. Novices to 3d puzzlers might end up getting confused and turned around in the dark, but seasoned game players will find it ridiculously easy. On the whole, I thought the game was... okay.. Definitely not as scary as the original, but the game was well made albeit short, the voice acting was good, and the music was excellent. I'd say it's a shame it's so short, but tbh, I was tiring of it by about hour 4, so I'm glad I finished it in under 5 hours. I don't know who I'd recommend it to... People that enjoy grisly movies and horror books, I suppose? Not your average gamer, though.
     [803] │ ---{ Graphics }--- ☐ You forget what reality is ☐ Beautiful ☑ Good ☐ Decent ☐ Bad ☐ Don‘t look too long at it ☐ MS-DOS ---{ Gameplay }--- ☐ Very good ☐ Good ☑ It's just gameplay ☐ Mehh ☐ Watch paint dry instead ☐ Just don't ---{ Audio }--- ☐ Eargasm ☐ Very good ☑ Good ☐ Not too bad ☐ Bad ☐ I'm now deaf ---{ Audience }--- ☐ Kids ☑ Teens ☑ Adults ☐ Grandma ---{ PC Requirements }--- ☐ Check if you can run paint ☐ Potato ☑ Decent ☐ Fast ☐ Rich boi ☐ Ask NASA if they have a spare computer ---{ Difficulty }--- ☐ Just press 'W' ☑ Easy ☐ Easy to learn Hard to master ☐ Significant brain usage ☐ Difficult ☐ Dark Souls ---{ Grind }--- ☑ Nothing to grind ☐ Only if u care about leaderboards ranks ☐ Isn't necessary to progress ☐ Average grind level ☐ Too much grind ☐ You'll need a second life for grinding ---{ Story }--- ☐ No Story ☐ Some lore ☐ Average ☑ Good ☐ Lovely ☐ It'll replace your life ---{ Game Time }--- ☐ Long enough for a cup of coffee ☑ Short ☐ Average ☐ Long ☐ To infinity and beyond ---{ Price }--- ☐ It's free! ☑ Worth the price ☐ If it's on sale ☐ If u have some spare money left ☐ Not recommended ☐ You could also just burn your money ---{ Bugs }--- ☐ Never heard of ☑ Minor bugs ☐ Can get annoying ☐ ARK: Survival Evolved ☐ The game itself is a big terrarium for bugs ---{ ? </> 10 }--- ☐ 1 ☐ 2 ☐ 3 ☐ 4 ☐ 5 ☑ 6 ☐ 7 ☐ 8 ☐ 9 ☐ 10
     [874] │  Amnesia: A Machine For Pigs: 100% COMPLETION REVIEW If you love completing games, take a look at our curator site. Amnesia: A Machine For Pigs pulls off the tone and atmosphere it is going for incredibly well. However, by playing the game, it is easily noticeable that it wasn't made to compete with its predecessor, The Dark Descent, but rather made as its own thing. The concept is entirely different: the true horror of A Machine For Pigs comes in lore and world building. I managed to find enough time to read every note in the game, and could play uninterrupted. I personally recommend the game, but it's definitely not a game for short attention spanned people, since the horror is subtle and requires willingness to read notes to be truly understood. Achievements are easy to unlock, check this guide: https:</></>steamcommunity.com sharedfiles filedetails</>?id=2981456003
     [888] │ This is a major downgrade from Dark Descent DD. - Feels more like a walking simulator, It's TOO LINEAR. - Infinite lantern - No inventory, no "Tinderbox"</>light material - Puzzles are way too easy and obvious, if you can call them puzzles at all. - Monster is just a hunchback pig humanoid - PAPA PAPA! DADDY! DADDY! - Maybe too many light sources already in place? I remember way back in time, A:DD was the peak of survival horror games. And still is nowadays, but this was a major flop. To be honest, I entered the factory and didn't felt like playing more, cause It's not scary and or imemrsive at all for me.
     [934] │ Got a cat here. Friends passing by can touch her and click Like to touch her once. {\u3000\u3000\u3000\u3000} {\u3000\u3000} ＿＿ {\u3000\u3000\u3000} {\u3000\u3000}／＞{\u3000\u3000}フ {\u3000\u3000\u3000} {\u3000\u3000}| {\u3000}_{\u3000} _ l {\u3000} {\u3000\u3000} {\u3000}／` ミ＿xノ {\u3000\u3000} {\u3000} </>{\u3000\u3000\u3000} {\u3000} | {\u3000\u3000\u3000} </>{\u3000} ヽ{\u3000\u3000} ﾉ {\u3000} {\u3000} {\u3000\u3000}|{\u3000}|{\u3000}| {\u3000}／￣|{\u3000\u3000} |{\u3000}|{\u3000}| {\u3000}| ￣ヽ＿_ヽ_ {\u3000}二つ
     [942] │  ---{ Gameplay }--- ⭐⭐⭐ Higher = great gameplay aka quite lot of fun ---{ Audio + Soundtrack }--- ⭐⭐⭐⭐⭐ Higher = Great audio and soundtrack ---{ Story }--- ⭐⭐⭐⭐⭐⭐ Higher = Better Story ---{ Difficulty }--- ⭐⭐⭐⭐⭐⭐⭐⭐⭐ Lower = Hard difficulty in this game ---{ Length of Content Inside this game }--- 🔲 What that was tiny. under 1 hour - 1 hour ✅ Short. 2-5 hours 🔲 Medium. 6-15 hours 🔲 Long. 16-48 hours 🔲 Super long. 49-250 🔲 Here's my keys, Philosophy, A freak like me, Just needs infinity. ---{ Bugs and Problems }--- Bugs ⭐ Lower = Smooth though out. Problems ⭐⭐⭐ Lower = No problem at all, you can play without it annoying you aka can play in peace. ---{ Price }--- £16.75 this is here just in case the full price gets lowered or even raised with time just so you know what it was priced at the time of this review was made 🔲 Worth the set price. 🔲 Small % off would do, aka around 5-30% range. 🔲 Medium % off would do, aka around 40-65% range. ✅ Large % off would do, aka 70% or higher off other wise you've just wasted your cash on this game. ---{ Detail Notes }--- This game was a bore to play pun intend too, it doesn't even come close to the first amnesia they even ripped out features. So no, you don't really hide or need to save oil or well anything, the only reason to look into desk draws and such is just story papers to read and collect that is all the lamp you carry infinite unlike the first. This didn't really scare me either it was so predictable, like oh i see a mail tube, i guess it wants me to use it cause there be a chase so it wants me to be able to open a shut door in the chase, oh what you know the door i came in has gone and there is a only a one way hallway i wonder what will happen and as the clear signs for told a small chase began. Also the only time i felt in danger got debunked quite fast the AI is even worse then the original game. Pig monster see me between a crate, i duck hoping i could of gotten away with it, what does it do keeps running into the crate not knowing it can just go around to get me, i stand up and witness what i thought was a threat as a stupid pig. For real the game isn't that fun, the only thing i can appreciate is the story to game is good enough, its for sure a improvement from the first more darker in tone but even more well written, but thats all this game has going for it. I know i got this game for free as this review is labled rightly so but, i would really hated spending £16 or even £7 for this, if you wanna buy it still, spite all its flaws of the lacking fun gameplay then best of luck on enjoying it. ---{ Verdict Rating ? </> 10 }--- {\t}⭐⭐⭐ {\t} 3 10
    [1023] │ I was expecting a horror game with a good storyline something like SOMA. And i got a walking simulator with a terrible enemy encounters. It's not a 'bad' game. The story was kinda interesting, but not enough to keep me entertained and i felt like even though it was only 6h long i want it to finally end so i can stop playing, cause it's not interesting as a game you mostly wonder corridors, do very basic "puzzles"</>turn knobs.
    [1130] │ A lot of stuttering for whatver reason =</> Similar enemies and gameplay as the first game, some mechanics were cut out, no new ones.
    [1131] │ Amnesia's Lackluster Sequel Amnesia: A Machine for Pigs is a survival horror game that was originally meant to be a mod. The game is an indirect sequel to Amnesia: The Dark Descent, which was both developed and produced by Frictional Games. While set in the same universe as the previous game, it features a new cast of characters and time setting. Follow My Review Group Here! About Description 💾Graphics The graphics are pretty good in this game. The atmosphere is incredible here with some interesting locations that change it up from the previous Amnesia game. The Monster Design is not that intimidating or scary in my opinion. The lighting is pretty good and you still have that annoying motion blur from the previous game which you can turn off luckily. 🎮Gameplay The gameplay here is watered down from the last game which sucks. You have infinite use of the lantern now so no more worrying about oil. I don’t like this feature at all. I enjoyed worrying about my oil consumption. It gave me something to think about and brought a challenge. The inventory system is gone. Just wiped away for no reason, which I question why they did that. The puzzles are barely even puzzles. You barely even interact with the monster. Even if you do, it's not much of a threat. It’s hard to find anything good to say about the gameplay. I guess you love horror games that have no fear or threat to you, then you can play this one. 📜Story I actually really enjoyed the story in this game. I love the twist later into the story. I feel like this is more of a rails on story game than a horror game. I highly recommend reading the notes since that will help you understand the game better as it can be confusing if you just skip them. 🕛Playtime This game took me like 5 hours to beat. It’s a one time only play for me. So don’t expect replayability here. 🎧Sound The voice acting is really good in this game. They did such a good job with making it sounds from the late 1800’s to early 1900’s. The environments can make some pretty creepy sounds as well which I really enjoyed. The music is pretty good too better than the first game. ❗Bugs❗ There is a huge bug in this game where if there is any trigger of an event, your frame rate will go down to like 30fps for a good few minutes. It takes me out of the experience every time this happens and I almost gave up on this game because of it. If you go to your journal and exit and can fix it, but not all the time. All and this happened a lot so be prepared for that. 🔅Performance Besides the near game breaking performance bug. I do get 60fps on max settings when the game wants to cooperate with me. ✔️Pro’s Amazing atmosphere Environmental Storytelling Some cool locations Good story Amazing voice acting ❌Con’s Annoying Micro Stutters at times Frame rate drops when events trigger Poor Optimization No more inventory system Puzzles are far too easy Almost no threats in this game Had to force Vsync on Nvidia Control Panel Terrible motion blur You can turn it off luckily 🏆Rating 3 10 Get this game on sale for around 75% off. Also don’t feel bad if you miss on the sale, you are not missing too much on this game. 🔗Conclusion This game was a huge disappointment. With the lack of interaction from the previous game and the lack of challenge this game has, I would just straight up skip this game. You are not missing anything ground breaking or amazing. The performance bug was horrible and the gameplay was just boring since there is rarely a threat. Move on and find a different game. It has a good story and scary environment and atmosphere, but that's not enough to make it worth buying for me especially coming from an incredible prequel to this. https:</></>store.steampowered.com app 239200 Amnesia_A_Machine_for_Pigs</> Links to help you in Amnesia: A Machine for Pigs: Amnesia: A Machine for Pigs - General Guide Master Archivist Achievement Guide Master Archivist: Maps Specs for Review: GPU: ASUS GeForce GTX 1070 8GB ROG Strix CPU: Intel Core i5-6600K 3.9 GHz RAM: Corsair Vengeance 16GB DDR4 3200MHz 
    ... and 2199 more
