Activity 8
================
Nathan Funckes

## Data and packages

Load the entirety of the `{tidyverse}`. You will do a little work with
dates in this activity so also load `{lubridate}`. Be sure to avoid
printing out any unnecessary information and give the code chunk a
meaningful name.

``` r
library(tidyverse)
```

Cheatsheets that you might want to add to your collection:

-   [`{stringr}`](https://stringr.tidyverse.org/index.html)
-   [`{forcats}`](https://forcats.tidyverse.org/)

Using `here::here`, upload the `billboard_songs.txt` file that is saved
in your `data/` folder. Notice that this file is a tab-delimited file
that is stored as a `.txt` file. Therefore, you will need to use either
`read_delim` with a `delim = ...` argument or (better) `read_tsv`.
Assign the file to a meaningful object name, be sure to avoid printing
any unnecessary information, and give the code chunk a meaningful name.

``` r
billboard_songs <- read_tsv(here::here("data","billboard_songs.txt"))
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   title = col_character(),
    ##   artist = col_character(),
    ##   `overall peak` = col_double(),
    ##   `weeks on chart` = col_double(),
    ##   `chart date` = col_double()
    ## )

These data include information on song popularity. In the US, the
Billboard Hot 100 is a list that comes out every week, showing the 100
most played songs that week. More information about the creation of this
dataset, as well as some analyses by the author, can be found here:
<https://mikekling.com/analyzing-the-billboard-hot-100/>. The dataset
you are provided is a limited version of the full data, containing:

-   `title`
-   `artist`
-   `overall peak`: The highest rank the song ever reached (1 is the
    best)
-   `weeks on chart`: The number of weeks the song was on the chart
-   `chart date`: The latest date the song appeared on the Billboard Hot
    100

This is a long dataset (34,605 observations)!

## Analysis

### Cleaning date

When you look at `chart date` you’ll notice that it was read in as a
numeric (`<dbl>`) variable type. What does the format of the date appear
to be in?

**Response**: double

In this activity, we will be using `str_...` functions to split or find
patterns in strings of information. Do the following with your dataset:

-   *Create* a variable called `date` that treats `chart date` *as a
    character* variable type,
-   Use `stringr::str_sub` on `date` to create three new columns:
    `year`, `month`, `day`
-   Replace/overwrite the variable `date` with an updated format using
    `lubridate::make_date`.

Once you have verified that this worked, overwrite your data object.

``` r
billboard_songs$date <- as.character(billboard_songs$`chart date`)
billboard_songs$date <- lubridate::make_date(substring(billboard_songs$date,0,4),substring(billboard_songs$date,5,6),substring(billboard_songs$date,7,8))
billboard_songs
```

    ## # A tibble: 34,605 x 6
    ##    title      artist     `overall peak` `weeks on chart` `chart date` date      
    ##    <chr>      <chr>               <dbl>            <dbl>        <dbl> <date>    
    ##  1 Uptown Fu… MARK RONS…              1               15     20150307 2015-03-07
    ##  2 Thinking … ED SHEERAN              2               20     20150307 2015-03-07
    ##  3 Love Me L… ELLIE GOU…              3                7     20150307 2015-03-07
    ##  4 Sugar      MAROON 5                4                6     20150307 2015-03-07
    ##  5 Take Me T… HOZIER                  2               28     20150307 2015-03-07
    ##  6 FourFiveS… RIHANNA &…              4                5     20150307 2015-03-07
    ##  7 Blank Spa… TAYLOR SW…              1               17     20150307 2015-03-07
    ##  8 Style      TAYLOR SW…              8                9     20150307 2015-03-07
    ##  9 Earned It… THE WEEKND              9                9     20150307 2015-03-07
    ## 10 Lips Are … MEGHAN TR…              4               18     20150307 2015-03-07
    ## # … with 34,595 more rows

#### Analyzing using dates

What 10 songs spent the longest on the charts? Give only the title,
artists, and weeks.

``` r
billboard_songs%>%
  top_n(10,`weeks on chart`)%>%
  select(c("title","artist","weeks on chart"))%>%
  arrange(desc(`weeks on chart`))
```

    ## # A tibble: 10 x 3
    ##    title                        artist                          `weeks on chart`
    ##    <chr>                        <chr>                                      <dbl>
    ##  1 Radioactive                  IMAGINE DRAGONS                               85
    ##  2 I'm Yours                    JASON MRAZ                                    76
    ##  3 How Do I Live                LeANN RIMES                                   69
    ##  4 Counting Stars               ONEREPUBLIC                                   68
    ##  5 Party Rock Anthem            LMFAO featuring LAUREN BENNETT…               67
    ##  6 Foolish Games / You Were Me… JEWEL                                         65
    ##  7 Rolling In The Deep          ADELE                                         64
    ##  8 Before He Cheats             CARRIE UNDERWOOD                              64
    ##  9 Ho Hey                       THE LUMINEERS                                 62
    ## 10 Need You Now                 LADY ANTEBELLUM                               60

**Response**:

What date did the oldest song(s) in this dataset leave the charts? Give
only the *distinct* dates.

``` r
billboard_songs%>%
  arrange(desc(date))
```

    ## # A tibble: 34,605 x 6
    ##    title      artist     `overall peak` `weeks on chart` `chart date` date      
    ##    <chr>      <chr>               <dbl>            <dbl>        <dbl> <date>    
    ##  1 Uptown Fu… MARK RONS…              1               15     20150307 2015-03-07
    ##  2 Thinking … ED SHEERAN              2               20     20150307 2015-03-07
    ##  3 Love Me L… ELLIE GOU…              3                7     20150307 2015-03-07
    ##  4 Sugar      MAROON 5                4                6     20150307 2015-03-07
    ##  5 Take Me T… HOZIER                  2               28     20150307 2015-03-07
    ##  6 FourFiveS… RIHANNA &…              4                5     20150307 2015-03-07
    ##  7 Blank Spa… TAYLOR SW…              1               17     20150307 2015-03-07
    ##  8 Style      TAYLOR SW…              8                9     20150307 2015-03-07
    ##  9 Earned It… THE WEEKND              9                9     20150307 2015-03-07
    ## 10 Lips Are … MEGHAN TR…              4               18     20150307 2015-03-07
    ## # … with 34,595 more rows

**Response**: 2015-03-07

### Artists who work together

Before we get to far, let’s create common definitions:

In the string below, Nicki Minaj and Young Thug are considered to be
**featured**.

    RAE SREMMURD featuring NICKI MINAJ & YOUNG THUG

In the string below, Jessie J and Ariana Grande and Nicki Minaj are all
considered to have **collaborated** on the song.

    JESSIE J, ARIANA GRANDE & NICKI MINAJ

Which artist has been **featured** on the most Billboard charting songs?

First, create a vector object called `artist_credentials` that takes
your dataset and *then* `pull`s only the `artist` column. The `pull`
function is like `select` except that it outputs a vector rather than a
data frame (tibble). Using this function will help us avoid many
“`coercing`” warnings - I find it easier to work with vectors of
strings.

Using the `artist_credentials` vector, do the following. I recommend
that you do one bullet at time and verify that it worked.

-   Use `str_subset` to pick only the observations that only include
    `"featuring"`,*then*
-   Use `str_replace_all` to remove everything before and including
    “featuring” for each of these observations (i.e., replace it with
    `""`), *then*
-   Use `str_split` to separate entries with multiple featured artist -
    these could include `,`, `&`, `or` (note the whitespace), or `and`
    (note the whitespace) (this should produce a list), *then*
-   Use `unlist` to lengthen all featured artists into one vector
    (compare the output here to the output from the previous bullet
    point), *then*
-   Use `str_trim` to remove whitespace from `"both"` sides of each
    entry.

When you verify that this works, save this information into an object
called `feat_artist`.

``` r
artist_credentials <- billboard_songs%>%
pull(artist)

feat_artist <- artist_credentials%>%
  str_subset("featuring")%>%
    str_replace_all(".*featuring ","")%>%
      str_split(",|&|or")%>%
        unlist()%>%
          str_trim()

sort(table(feat_artist),decreasing=TRUE)[1]
```

    ## LIL WAYNE 
    ##        69

Now you can use this information to answer the question above.

**Response**: LIL Wayne

Which artist has **collaborated** on the most Billboard charting songs?

Using the `artist_credentials` vector you created before, do the
following:

-   Use `str_replace_all` to remove everything after and including
    “featuring” for each of these observations (i.e., replace it with
    `""`), *then*
-   Use `str_extract_all` to separate entries by `,` or `&`, *then*
-   Use `unlist` to lengthen all collaborating artists into one vector,
    *then*
-   Use `str_trim` to remove whitespace from `"both"` sides of each
    entry.

When you verify that this works, save this information into an object
called `collab_artist`.

Now you can use this information to answer the question above.

#### Challenge 1

What songs could have been played at your 16th birthday party? That is,
which songs that eventually peaked at \#1 **entered** the charts within
a couple months (before or after) your 16th birthday? Give only the song
title, artist, and date of chart entry.

Note that this might not be possible depending on when your 16th
birthday occurred (i.e., if you turned 16 after March 7, 2015 - I need
to find more recent songs :/).

**Response**:

## Challenge 2: Data Exploration

This is somewhat unrelated to the steps you completed in the rest of
this activity, but you have added experience with new tools that you may
find useful.

Create some visual representation or summary table of these data. I
would encourage you to explore how you can use `{forcats}` to help make
your visual/table more human-readable/effective for telling the story of
your exploration.

## Attribution

Parts of this Activity are based on a lab from [Dr. Kelly
Bodwin’s](https://www.kelly-bodwin.com/) STAT 331 course.
