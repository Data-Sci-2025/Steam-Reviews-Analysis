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


## October 25, 2025

Have downloaded reviews for my 45th game to replace the multiplayer game I'd previously included by accident. (Postal 3 (Mostly Negative))

I solved the issue of naming the column that automatically populated with the game's unique Steam upload ID number by renaming by column number instead (it is always column number 2)
   - I still want to preserve the steam ID number and will add it to the dfs with the games' basic information, just so it's saved. 
   
I also solved the issue of the review_type column being completed with emojis (green check for positive red x for negative) with text (POS, NEG) using str-Replace_all(), quick and easy!
   
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




