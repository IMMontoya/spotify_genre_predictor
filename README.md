# Spotify Genre Predictor - Data Gathering

## Table of Contents

- [Video Presentation](https://youtu.be/wWbh7Tuw_cM)
- [Notebook](/Pudding.ipynb)
- [Project Overview](#project-overview)
- [Link to Original Repository](https://github.com/mtgleason/Sept_2024_Code_Jam)
- [Data Gathering - My Contribution](#data-gathering---my-contribution)
- [Conclusion](#conclusion)
- [Installation](#installation)

## Project Overview

This repository details my contributions to the collaborative [September 2024 Code Jam Project](https://github.com/mtgleason/Sept_2024_Code_Jam) hosted by [TripleTen](https://www.youtube.com/watch?v=PrxnZ_-yPd0).

### Team Members

- [Michael Velasco](https://github.com/vekim91)
- [Isaiah Montoya](https://github.com/IMMontoya)
- [Michael Gleason](https://github.com/mtgleason)

### Objective

Build a machine learning model to predict the genre of a song, using data searched from the [Spotify Web API](https://developer.spotify.com/documentation/web-api).

### Scope

In order to create a minimum viable product (MVP) in a short amount of time, the team decided to focus on a small number of genre labels:

1. Rock
2. Pop
3. Rap/Hip-Hop
4. Classical
5. Jazz
6. Other

## Data Gathering - My Contribution

Video Presentation detailing the data gathering process:  
[![Watch the presentation](https://img.youtube.com/vi/wWbh7Tuw_cM/0.jpg)](https://youtu.be/wWbh7Tuw_cM)

### Super / Sub Genres

Sub-genres were defined for each genre label within the dictionary `genres_of_interest`. This resulted in three major benefits:

1. **Larger dataset:** The Spotify API limits results per genre search to 900. By grouping multiple sub-genres under a single genre label, we were able to gather more data for each genre.

2. **More wholistic range of features per genre:** By including multiple sub-genres under a single genre label, we were able to gather a more diverse range of features for each genre, while still maintaining a clear distinction between genres.

3. **Narrower range of features for the other label:** Because the "Other" label was defined as a catch-all for genres not included in the genres of interest dictionary, it was important to insure sub-genres more related to the genres of interest were not included in the "Other" label.

### Genres of Interest Dictionary

```python
genres_of_interest = {
    'rock': [
            'rock',
            'alt-rock',
            'hard-rock',
            'j-rock',
            'psych-rock',
            'punk-rock',
            'rock-n-roll',
            'rockabilly',
            'grunge',
            'punk'
            ],
    'pop': [
            "pop",
            "Dance Pop",
            "Electropop",
            "Indie Pop",
            "Synth-pop",
            "Pop Rock",
            "Teen Pop",
            "Power Pop",
            "Art Pop",
            "Pop Punk",
            "K-Pop",
            "J-Pop",
            "Latin Pop",
            "Dream Pop",
            "Bubblegum Pop",
            "Euro Pop",
            "Pop Rap",
            "Chamber Pop",
            "Baroque Pop",
            "Pop Soul",
            "Acoustic Pop",
            "j-pop",
            "k-pop",
            ],
    'rap/hip-hop': [
                "Hip Hop",
                "Hip-Hop",
                "Rap",
                "Trap",
                "Gangsta Rap",
                "East Coast Hip Hop",
                "West Coast Hip Hop",
                "Conscious Hip Hop",
                "Alternative Hip Hop",
                "Boom Bap",
                "Dirty South",
                "Crunk",
                "Drill",
                "Grime",
                "Cloud Rap",
                "Underground Hip Hop",
                "Emo Rap",
                "Hardcore Hip Hop",
                "Lofi Hip Hop",
                "Old School Hip Hop",
                "Christian Hip Hop",
                "Latin Hip Hop"
                ],
    'classical': [
                "Classical",
                "Baroque",
                "Romantic",
                "Classical",
                "Chamber Music",
                "Symphony",
                "Opera",
                "Choral",
                "Contemporary Classical",
                "Minimalism",
                "Orchestral",
                "Piano",
                "String Quartet",
                "Early Music",
                "Renaissance",
                "Modern Classical",
                "Neoclassical",
                "Impressionism",
                "Avant-Garde",
                "Sacred Classical",
                "Cantata",
                "Piano"
                ],
    'jazz': [
                "Jazz",
                "Bebop",
                "Swing",
                "Smooth Jazz",
                "Cool Jazz",
                "Hard Bop",
                "Free Jazz",
                "Fusion",
                "Modal Jazz",
                "Latin Jazz",
                "Avant-Garde Jazz",
                "Gypsy Jazz",
                "Vocal Jazz",
                "Jazz Funk",
                "Jazz Blues",
                "Soul Jazz",
                "Post-Bop",
                "Ragtime",
                "Big Band",
                "Dixieland",
                "Nu Jazz",
                "Jazz Fusion",
                ]
}
```

The `genres_of_interest` dictionary can easily be modified to include additional sub-genres or to change the genre labels. This flexibility allows for easy expansion of the dataset and the ability to gather data for different genre labels. The `test_genre_string` function and the `ammend_sub_genres` function can be used to test and modify the dictionary respectively.

```python
# This validates each string in the lists per super genre. If the string is not a recognized genre, it gets removed from the super genre list - preventing unnecessary API calls.

for super_genre in genres_of_interest:
    print(f"\nChecking sub-genres for {super_genre}")
    genres_of_interest[super_genre] = amend_sub_genres(genres_of_interest[super_genre])
```

### API Calls

#### Get Genres of Interest

Once the `genres_of_interest` dictionary is validated, the `get_genres_of_interest` function can be used to gather track ids for each genre label. This function will make API calls for each sub-genre in the dictionary as evenly as possible, instances where there cannot be an equal number of records per sub-genre are handled by randomizing the order of the sub-genres before making the API calls. Returns a tuple of lists containing the track ids and the genre label for each track.

#### Get Other Genre

The `get_other_genres` function defines what sub-genres to consider as "Other" by flattening the `genres_of_interest` dictionary then applying list comprehension to the `sp.recommendation_genre_seeds()` to create a list of available genres not included in the `genres_of_interest` dictionary. This list is then used to make API calls for the "Other" genre label in a similar fashion to the `get_genres_of_interest` function. Returns a tuple of lists containing the track ids and the genre label for each track.

#### Get Audio Features

The tuples for then combined to create a dataframe of track ids and genre labels. The audio features are then retrieved for each track id using the `sp.audio_features()` method. The audio features are then combined with the genre labels and track ids to create a dataframe.

```python
# Combine the two lists
all_track_ids = track_ids + other_track_ids
all_track_genre = track_genre + other_genre_labels

# Create a DataFrame
track_genres_df = pd.DataFrame({"track_id": all_track_ids, "genre": all_track_genre})

# Get audio features for each track with a progress bar
track_features = []
for i in tqdm(range(0, len(track_genres_df), pagination_limit), desc="Fetching audio features"):
    features = sp.audio_features(all_track_ids[i:i+pagination_limit])
    
    # Raise error if no features are returned
    if not features:
        raise ValueError(f"No audio features returned for tracks: {all_track_ids[i:i+pagination_limit]}")
    
    for feature in features:
        # Raise error if no features are returned for individual tracks
        if not feature:
            raise ValueError(f"No audio features returned for track: {all_track_ids[i:i+pagination_limit]}")
        track_features.append(feature)

# To DataFrame
track_features_df = pd.DataFrame(track_features)

# Merge the two DataFrames
all_data = pd.merge(track_genres_df, track_features_df, on='track_id')
```

#### Get Artist and Album Information

The `get_artist_album_info` function is used to gather artist and album information for each track. This information is then combined with the audio features and genre labels to create a final dataframe.

```python
# Get artist and album data
artist_album_df = get_artist_and_album(all_data['track_id'].tolist())

# Merge artist and album data
all_data = pd.merge(all_data, artist_album_df, on='track_id')

# Save the data
all_data.to_csv('spotify_data.csv', index=False)
```

## Data

**Description of the spotify_data.csv:**

- `track_id` - Unique ID for each track
- `genre` - song genre (Target)
- `danceability`
- `energy`
- `key`
- `loudness`
- `mode`
- `speechiness`
- `acousticness`
- `instrumentalness`
- `liveness`
- `valence`
- `tempo`
- `type`
- `uri`
- `track_href`
- `analysis_url`
- `duration_ms`
- `time_signature`
- `artist`
- `album`

## Conclusion

The data gathering process was successful in gathering a diverse range of audio features for each genre label. The data was saved to a CSV file for further analysis and model building to be done by the other team members.

The flexibility of the `genres_of_interest` dictionary allows for easy expansion of the dataset and the ability to gather data for different genre labels. The `test_genre_string` function and the `ammend_sub_genres` function can be used to test and modify the dictionary respectively.

Click [here](https://github.com/mtgleason/Sept_2024_Code_Jam) to view the original repository for this group project.

## Installation

Follow these steps to set up the project locally on your machine.

### 1. Clone the Repository

First, clone the repository to your local machine using Git.

```bash
git clone https://github.com/IMMontoya/spotify_genre_predictor.git
```

### 2. Create a Virtual Environment

```bash
python3 -m venv venv
```

### 3. Activate the Virtual Environment

Mac/Linux:

```bash
source venv/bin/activate
```

Windows:

```bash
venv\Scripts\activate
```

### 4. Install the Required Libraries

```bash
pip install -r requirements.txt
```

**Note:** Python 3.10.12 was used for this project.

### Usage

A software able to read [Jupyter Notebook](https://jupyter.org/) files such as [VS Code](https://code.visualstudio.com/) is necessary.

Visit the [Spotify Web API](https://developer.spotify.com/documentation/web-api) website, follow instructions to recieve API ID and secret.

Place them in a file named `.env` using the format:

``` md
Client_ID = 'ID_HERE'
Client_secret = 'secret_HERE'
```
