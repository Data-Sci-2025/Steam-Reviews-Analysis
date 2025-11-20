# Progress Report

## October 4, 2025

At this point I have identified 5 games per Steam rating ranking (45 total games) and downloaded reviews for almost all of them (44/45). One game was later discovered to not meet my criteria and will be replaced. 

Games were selected with the following criteria in mind:
- Single-player games (primarily)*
- older than 1 or 2 years (where possible)
- Not AAA studios (filtered roughly by selecting by price)
- Price range =<$40 (not worried about minimum price but >$10 ideal)
- Has at least 1000 number of reviews (where possible)
- Game in English, only considering English reviews
- Full games only (no downloadable content), sequels OK
- No limitations on genre or game play style, actively seeking variety
- only select games that the overall review ranking and the english review ranking are the same

*if a single player game has optional/additional multiplayer function, that's fine.

I tried to avoid games that are abandoned, but the options in the lower rankings were pretty limited and I couldn't entirely avoid them. 

1. I've built a df of all the games and their basic information, title, year, ranking, percentage of positive reviews, number of reviews & of english reviews, and price. I don't know if it will be used ultimately but it's key information to the titles selected.

2. I'm experimenting with the data cleanup process so that I can create a workflow code for all of the data. Ideally, the same code can be reused across all files to read the data into a df with only the information needed. 

For the most part, this is straightforward. Renaming columns from their default values first. One potential snag here is that it appears one of the columns auto-populates with the game's Steam ID number, which is used to pull reviews. I've not yet decided what to do with that column, or how best to deal with it iteratively across the different data files. 

Some columns populated with emoji-style data that I would like to full replace with a text value. Still a lot of experimentation to do with the data formatting and what I want to keep around and what to do with it. 

3. I have a data disparity. Because of how the games are ranked, it's not only review positivity that plays a role in a game's overall ranking in Steam, but also number of reviews. This particularly comes up in the "positive" and "negative" elements (3rd and 7th in the overall 9 pt scale). A number of these games were ranked 100% positive (or negative) but only have up to 50 total reviews. At 51 reviews they would, it seems, be boosted up to "Very" positive or negative on the scale. 

Negatively ranked games also have fewer total reviews in general. It's unlikley after a certain point, I suppose, that someone would see a game with a few hundred negative reviews and decide to give a game a shot anyway. 

I don't know yet what this is going to mean for how many reviews I pull for each game rank, it's something to think about more in depth.


## October 25, 2025 - Progress Report 1

Have downloaded reviews for my 45th game to replace the multiplayer game I'd previously included by accident. (Postal 3 (Mostly Negative))

I solved the issue of naming the column that automatically populated with the game's unique Steam upload ID number by renaming by column number instead (it is always column number 2)
   - I still want to preserve the steam ID number and will add it to the dfs with the games' basic information, just so it's saved. 
   
I also solved the issue of the review_type column being completed with emojis (green check for positive red x for negative) with text (POS, NEG) using str-Replace_all(), quick and easy!


### Next Steps

I'm working on building a function to read in every file in my data file collection (all .csv files). I want it to:

1. read in all files (datafiles/*.csv)

2. take steam ID # from column 2 and add it to a new column (it'll be repetitive but a good merge point if I decide to join my other game dfs)

3. rename column 2 (to language)

4. concatenate each file into one giant df of game data

I think this is feasible but I've not successfully created a function before, so this is my big trial and error at this point. I could just do it manually one by one but this would be much smoother!

Finally, I've been expanding on some plans thanks to our latest reads in the tidytext chapters. I already talked about tokenizing the reviews and looking at some information that way, but I have some more clearly defined ideas since then. 

- I want to look at word types and word tokens (and TTR)
- do some exploratory looks (avg review length, language used, maybe some others)
- probably some quick stats (correlation at least)
- tf-idf, are there non-stop words that are extremely common? some visualizations, all primarily looking at review type (positive or negative) 

Then, of course, the classifier ultimately. 


## November 11, 2025 - Progress Report 2

First, I fixed my output for my "data exploration" .qmd file. Sorry to Sarah for making you read through that instead of a nice tidy .md on github! It's working properly now and looks nicer. 

Another note from Sarah's comment in my guestbook was about how I got the information about a game's review ranking on the positive/negative scale and I hadn't even thought about that! I added a [notes and info](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/tree/main/notes_and_info) folder in my github (subject to change) with the output df of all the games I have reviews for and my projnotes.md file. Projnotes was really just for me to keep track of everything as I was working and thinking, but all of the information came directly from the Steam app which lists the number of reviews (total and in English if there are enough reviews), the % positive of of those reviews, the game's ranking, etc. The date listed is the date I downloaded the reviews just so I knew it, since new reviews are being posted constantly.

My final report will have notes about how these rankings are decided based on reviews, it's not as straightforward as I imagined it going into this project!

I got a dataframe built with my review data, though my steam_id column is still not populating correctly. I think it's related to my order of operations in my df building function, but I have to talk to Dan about it this week to puzzle it out.

I've moved ahead in the meantime and I'm knee deep in data cleanup. This process can be found in the "data cleanup" file in my [data_processing](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/tree/main/data_processing) sub-folder in github. It's in early stages, but I have my cleanup plans detailed in there as a checklist for me to follow step-by-step. Once I get the steam_id column corrected, it won't affect the cleanup pipeline in any way, it will just affect the 9th column in the dataframe (it's already there, just not populated correctly). With column 9 corrected, I'll overwrite the reviews_df.csv and it will read into the pipeline just the same. 

Once my data is cleaned up it'll be early analysis time!

### Sharing Scheme

I'm sharing (and already have) my raw data as it is all publicly available data anyway

### License Information

I'm choosing CC-BY-SA as my license. Since I'm using and reformatting publicly found data, I think it's fair that others are able to use my transformed data the same way.

### Pipeline options

I'm choosing to use the New Continuing option for my code pipeline, with my qmd files numbered in the order intended to be read in. I'll add a note about this in my qmd files as well.


## November 19, 2025 - Progress Report 3

I've gotten my data into the shape I want it to be in to work with! I still can't upload my dataframe because it's too big, but it's easy enough to recreate by following my data processing .qmd files and follow what I've done, or formulate something new and unique. 

Now that I'm here, I've gone full tilt into analysis, but have more work to do still. [Here](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md) is the latest I've accomplished. 

### What I've done

- Review counts total by game and by review type (next: bar chart)
- [Word count](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#word-count) aka review length, average review length, highest and lowest review lengths
- Review length [distributions](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#length-stats) (log transformed) (next: update violin plot)
- [Word types](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#word-types--count) and count
- [TTR](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#ttr). With such a majority of reviews being so short I don't see it being a useful metric, but I took a look still.
- [Tf-idf](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#tf-idf). This section needs the most tidying since I was messing with it, hopefully I'll get it a little tidier before tomorrow.

I've also created an additional notebook called "stretch goals". This is my kind of lofty ideas outlet. Things I'm interested in but are either not extremely linguistically relevant (like reviews posted over time) or are challenging enough I decided to set them aside for the more important goal of the classifier (emoji analysis)


### What I'm doing

I've snipped out a subset of my full data set and am putting together a classifier. Once I'm sure it's functioning on the small subset, I'll be clear to set the full sized data through it. 

I want to try to classify two things:

1. review positivity or negativity based on words used (tf-idf data) and review length

2. If I can, game rank, simplified down to positive, mixed, or negative, based on % of of positive vs negative reviews. I'll likely set this aside as a bonus goal since it's not exactly linguistically relevant. 

### What's next

1. my sample sizes are very very different. Positively reviewed games receive more game reviews than negatively reviewed games do. Check out the actual numbers [here](https://github.com/Data-Sci-2025/Steam-Reviews-Analysis/blob/main/data_processing/2-data-analysis.md#positive-and-negative-review-length). Because of this, comparative analysis has been pretty tricky. I'm going to use slice_data to cap the number of reviews for positive games so that things are a bit more even. 

2. Tidying my notebooks. I had to do some additional cleanup in my analysis notebook, and I plan to move it back to my cleanup notebook so things are all in their place. Some moving around of bits and section headers, mostly style and organization choices

3. Fix my visualizations as mentioned above. Once I do my downsampling all the numbers are going to change anyway, so I want to get the classifier functioning first and then throw my whole energy into the downsampling. 


### Questions in mind

For my classifier I plan to use word count (review length) as a predicting variable since it seems like, on average, negative reviews are a bit longer than positive ones. For that, should I used the 'word_count' column from may full reviews_df or the 'total' column from the tf-idf df which is the review total length after stop words are removed?



