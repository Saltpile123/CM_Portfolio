Computational Musicology Portfolio Plan
================
Leander van Boven - 12997080

# The idea

For my project I’m going to do research in the genre of hardstyle music.
A lot of people would say that the music within this genre is all alike.
However there is a common assumption that each artist distinguishes him-
(or her)self with his (or her) unique style and sound. This are most
noticable in the tones used in the so called *drop* and as bass-kick.

I’m going to research whether this assumption can be proved with a
(computer)model. In particular a classification model that can classify
a song with an artist (assumed that this song is of one of the artists
used for training the model). Because if such model can be used we can
assume that indeed there is something in the songs that are unique for
each artist. However if such model is not possible, I’m going to
research why this is the case, or what is necessary to create such
model.

# The data

To do this research we obviously need some data to work with. For this
I’m going to use the songs from the top 5 hardstyle artists together
with the songs of my 2 most favorite artists. These artists include:

| Artist             | Songs on Spotify |
| ------------------ | ---------------- |
| Noisecontrollers   | 199              |
| Headhunterz        | 146              |
| Brennan Heart      | 100              |
| Showtek            | 88               |
| Da Tweekaz         | 62               |
| Sub Zero Project   | 56               |
| D-Block & S-Te-Fan | 47               |

Together this gives us a corpus of 698 songs where each artist has about
50 songs or more. This should be enough data to build a decent
classification model.

# The planning

### Data understanding

Before trying to build a classifier we first need to do some exploration
on and understanding of the data. In the first place we need to decide
which information we are going to use for the classifier. For example
the genre for each artist probably will be similar and thus will not be
useful data. Furthermore we have two possible sets of features we can
use to train the classifier with:

  - *Track Features*, these features are returned by the
    `get_track_audio_features()` method of the `spotifyr` package. This
    method is also used to get the track features in the
    `get_artist_audio_features()` and `get_album_audio_features()`
    methods. These features are values that say something about the song
    in a whole, thus we will get 1 feature value per song.
  - *Track Analyis*, these features are obtained using the
    `get_track_audio_analysis()` method fromt the `spotifyr` package.
    The analysis features are quite a bit more extensive than the track
    features, thus will probably contain a lot more information about
    the song. However this means more data, which will take up more disk
    space, take longer to obtain from Spotify, make the classifiation
    training take more time and make the model quite a bit more complex
    (since we now need to add a time dimension to our model).

Because of the reasons described above I’m first going to focus on
creating a model created with the *track features*. If I fail to create
a good model with these features I’m going to take a look at the *track
analysis* features.

The track features include many features including the following numeric
features that may be useful:

  - Danceability
  - Energy
  - Key
  - Loudness
  - Mode
  - Speechiness
  - Accousticness
  - Instrumentalness
  - Liveness
  - Valence
  - Tempo

This is a lot of data in which some features may be very similar for all
songs. It is useless to include this data in the trainingsdata for the
classifier since it wouldn’t provide good information to distinguish two
songs from each other, let alone different artists.

To give a good insight in these features and get a quick overview of
which of these may be show some clear differences between the artists, I
have combined all songs from all artists into one dataset and plotted
each
feature:

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-2.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-3.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-4.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-5.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-6.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-7.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-8.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-9.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-10.png)<!-- -->![](README_files/figure-gfm/unnamed-chunk-1-11.png)<!-- -->

As you can see the *Mode* feature only has two values and no artist seem
to prefer one or another, thus we can discard this feature from our
dataset. Furthermore the *Danceability* / *Speechiness* / *Liveness* and
*Valence* plots show no clear patterns or differences between the
artists, thus are probably also not useful information to include in the
data. The *Key* histogram does show some big differences in key usage,
however this is key usage between songs and not between artists in
particular. All artists seem to prefer a key of \~1 or \~8.

But in for example the *Loudness* plot we can see some clear patterns
difference between the artists where the songs of another artist are in
the dataset. The *Energy* and *Tempo* plots look promising too.

-----

> All items below are a planning and description of things I’m going to
> do in the upcoming weeks, these are subject to change as result of
> feedback and/or new things learned during the course.

### Data Preparation

Beofre we can feed the data to the classifier we first need to prepare
the data. One part of data preparation is data reduction. This means
that we reduce the initial dataset to be only data we are going to use
for the classifier. Since we are going to use the track features we can
all discard all data other than these featuer. However as discussed
earlier the *Mode* feature does not include any useful information thus
we can discard that data as well. Of course we still need to include the
*Artist* in our reduced dataset since we need to use that data for as
classes for the model. As mentioned before most of the other features
didn’t show much patterns too, however since we can’t rule out that they
might be useful we still need to include it. This, however, may mean
that we get alot of information sparse data.

This is why I’m going to apply a principal component analysis on those
possibly information sparse data and use that data to create another
model. Principal component analysis means that we reduce the data to a
new dataset where each column is an information rich column that
captures as much possible variation from the initial data. This data may
be even better to use than the features on their own since the PCA data
will be more dense in information.

Since we are going to train a classifier we also need to seperate the
data into two subsets:

  - A trainingset, containing about 80% of the data. This data will be
    used to train the classifier.
  - A testset, containing the remaining 20% of the data. This data will
    be used to test the classifier.

However, since taking only 20% of the data as validation data, this
means we get only 140 songs to validate the classifier. In the best
scenario this will mean 20 songs per artist, however since we don’t have
an equal amount of songs per artist we will most likely not get 20 songs
per artist in the validation data.

For this reason I’m going to perform cross validation on the data with 5
batches. This means that I’m going to shuffle the data and then divide
the data into 5 parts (thus each part is 20% of the data). After that I
will repeatedly take one part as testset and the other parts as
trainingset. This way all data will be once testdata and 4 times
trainingsdata, resulting in a model that has seen more data and thus is
less overfitted on that data, hopefully giving a model that can classify
novel songs better.

### Modeling

As mentioned before I’m going to train multiple models, trained on
different subsets of the initial dataset:

  - A model trained on all features mentioned in the **Data
    Understanding** without the *Mode* feature.
  - A model trained on (a subset of) the Principal Components.
  - (Optionally) A model trained on (a subset of) the
    `get_track_audio_analysis()` features, if necessary.

Depending on the results of these models I may deduce what is causing
them to perform in their way, and build new models that may perform
better.

### Evaluation

For each model we can assess it’s performance by the amount of songs it
classified correctly. We can use an extensive confusion matrix to
compare the amount of correctly classified songs to the amount of
incorrect classifications to see which songs are related to each other.
After all, if the songs for a certain artist that were incorrectly
classified almost always were classified with a certain other artist,
those artists must be very similar to each other in terms of the
features used for the model.

Optimally we want to create a model that uses only the features returned
by the `get_track_audio_features()` method, without diving into the
sounds and tune of a song (the data returned by the
`get_track_audio_analysis()` method). I’m going to compare the different
models with each other and find which model performed best and why. If
two artists turn out to be very similar in each model, I may look for
the most similar tracks and subjectively compare them by for example
listening to them to see whether those tracks are indeed similar to the
ear as well.

For the future I hope that some of the artists used to build the model
will release new songs, thus allowing me to test the model on novel data
to see whether the model is indeed as good as it proved to be on the
testdata.

-----
