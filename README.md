# Spotify-Data-Engineering-Framework
# Spotify Data Analysis Project 🎵

## **Project Overview**
This repository demonstrates a comprehensive data engineering and analysis pipeline leveraging **Spotify’s Web API** and **Apache Spark**. The project focuses on extracting, processing, and analyzing data from Spotify’s Global Top 50 playlist. By employing a multi-layered data architecture (bronze, silver, gold), this project showcases best practices for handling real-time streaming data, performing complex transformations, and deriving actionable insights.

Key features include:
- Building a scalable **ETL (Extract, Transform, Load)** pipeline.
- Exploring **audio features** and **recommendation algorithms** to analyze music trends.
- Profiling artists based on popularity, audio features, and release timelines.
- Generating **data visualizations** and dashboards for insights.

---

## **Technologies Used**
- **Apache Spark**: Distributed data processing.
- **Spotify Web API**: Dynamic data ingestion.
- **Python**: Data transformation and orchestration.
- **Delta Lake**: Incremental data storage.
- **Databricks**: Collaborative workspace for data engineering.

---

## **Data Pipeline Architecture**
The project implements a layered data architecture:

1. **Bronze Layer**: Raw data ingestion from Spotify API.
2. **Silver Layer**: Data cleaning and enrichment.
3. **Gold Layer**: Aggregated datasets for business intelligence.

---

## **Technical Implementation**

### **1. Environment Setup**
Configuration values for API endpoints and credentials are stored in interactive widgets for dynamic access:
```python
# Spotify API Configuration
BASE_URL = "https://api.spotify.com/v1"
TOKEN_URL = "https://accounts.spotify.com/api/token"
PLAYLIST_ID = "37i9dQZEVXbNG2KDcFcKOF"
CLIENT_ID = "your_client_id"
CLIENT_SECRET = "your_client_secret"
```
Databases for layered storage are created using Spark SQL:
```sql
CREATE DATABASE IF NOT EXISTS bronze;
CREATE DATABASE IF NOT EXISTS silver;
CREATE DATABASE IF NOT EXISTS gold;
```

### **2. Spotify API Integration**
A custom class `SpotifyAPI` manages all API interactions:
- **Authentication**: Retrieves an access token using the client credentials flow.
- **Data Retrieval**: Fetches JSON data from endpoints like playlists, tracks, and recommendations.
- **Processing**: Converts API responses to Spark DataFrames.

Example method to retrieve playlist tracks:
```python
def get_playlist_tracks(playlist_id: str):
    response = SpotifyAPI().parse_json("playlists", f"{playlist_id}/tracks")
    return SpotifyAPI.read_json_to_df(response["items"])
```

### **3. Exercises Overview**
Each step in the pipeline corresponds to an exercise that builds on the previous one.

#### **Exercise 1: Global Top 50 Extraction**
- Extracts tracks from Spotify’s Global Top 50 playlist.
- Transforms raw data into a structured format.
- Saves data in the `bronze.global_top_50` table with the schema:
  ```
  |-- track_id: string
  |-- album_id: string
  |-- playlist_id: string
  |-- track_name: string
  |-- track_artists: array<string>
  |-- track_popularity: integer
  |-- track_duration_ms: integer
  |-- album_urls: map<string, string>
  |-- album_release_date: string
  |-- added_at: timestamp
  ```

#### **Exercise 2: Recommendation Engine**
- Fetches recommendations based on tracks from Exercise 1.
- Stores the output in `bronze.recomm_global_top_50`.
- Parameters for recommendations include:
  - `min_energy`: 0.5
  - `min_popularity`: 45
  - `max_popularity`: 75
  - `seed_genres`: indie, rock

#### **Exercise 3: Album Details**
- Retrieves detailed album information for tracks in the recommendation dataset.
- Combines album and track data for a unified view.
- Saves results in `bronze.recomms_full_detail_album`.

#### **Exercise 4: Audio Feature Enrichment**
- Enriches track data with audio features such as:
  - Danceability, energy, tempo, liveness, valence.
- Correlates audio features with track popularity.
- Stores data in `silver.recomms_full_detail_album_enriched`.

#### **Exercise 5: Artist Profiling**
- Aggregates audio features and popularity at the artist level.
- Calculates the time since the latest release per artist.
- Saves results in `gold.recomm_artists_profiling`.

Example aggregation:
```python
agg_df = (
    album_track_full_detail_df
    .groupBy("artist_name")
    .agg(
        F.avg("track_popularity").alias("avg_popularity"),
        F.avg("danceability").alias("avg_danceability"),
        F.max("release_date").alias("latest_release_date")
    )
    .withColumn("days_since_latest_release", 
                F.datediff(F.current_date(), F.col("latest_release_date")))
)
```

---

## **Running the Project**
### **Prerequisites**
- Python 3.8+
- Apache Spark
- Spotify Developer Account

### **Setup Instructions**
1. Clone the repository:
   ```bash
   git clone https://github.com/your-profile/spotify-data-analysis.git
   cd spotify-data-analysis
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Configure Spotify API credentials in `config.json`:
   ```json
   {
       "client_id": "your_client_id",
       "client_secret": "your_client_secret",
       "playlist_id": "37i9dQZEVXbNG2KDcFcKOF"
   }
   ```

4. Run the scripts in sequence or interactively in a Databricks environment.
