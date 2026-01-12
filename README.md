# Spotify Advanced SQL Project and Query Optimization P-6
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 4. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 5. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

## 15 Practice Questions

### Easy Level
1. Retrieve the names of all tracks that have more than 1 billion streams.
   ```sql
	SELECT
	TRACK
	FROM Spotify
	WHERE STREAM > 1000000000;
   ```
2. List all albums along with their respective artists.
   ```sql
	SELECT
	DISTINCT ALBUM,
	ARTIST
	FROM Spotify;
   ```
3. Get the total number of comments for tracks where `licensed = TRUE`.
   ```sql
	SELECT
	SUM(COMMENTS) AS Total_Comments
	FROM Spotify
	WHERE LICENSED = TRUE;
   ```
4. Find all tracks that belong to the album type `single`.
   ```sql
	SELECT
	TRACK,
	Album_Type
	FROM Spotify
	WHERE Album_Type = 'single';
   ```
5. Count the total number of tracks by each artist.
   ```sql
	SELECT
	ARTIST,
	COUNT(TRACK) AS Total_Tracks
	FROM Spotify
	GROUP BY ARTIST;
   ```

### Medium Level
1. Calculate the average danceability of tracks in each album.
   ```sql	
	SELECT
	ALBUM,
	AVG(Danceability) AS Average_Danceability
	FROM Spotify
	GROUP BY ALBUM
	ORDER BY Average_Danceability DESC;
   ```
3. Find the top 5 tracks with the highest energy values.
   ```sql
   	SELECT
	TRACK,
	ENERGY
	FROM Spotify
	ORDER BY ENERGY DESC
	LIMIT 5;
   ```
5. List all tracks along with their views and likes where `official_video = TRUE`.
   ```sql
	SELECT
	TRACK,
	VIEWS,
	LIKES
	FROM Spotify
	WHERE Official_Video = TRUE;
   ```
7. For each album, calculate the total views of all associated tracks.
   ```sql
   	SELECT
	ALBUM,
	TRACK,
	SUM(VIEWS) AS Total_Views
	FROM Spotify
	GROUP BY ALBUM, TRACK
	ORDER BY ALBUM, Total_Views;
   ```
9. Retrieve the track names that have been streamed on Spotify more than on YouTube.
   ```sql
	SELECT * FROM
	(SELECT
		TRACK,
		COALESCE(SUM(CASE WHEN Most_Played_On = 'Youtube' THEN STREAM END),0) AS Streamed_On_Youtube,
		COALESCE(SUM(CASE WHEN Most_Played_On = 'Spotify' THEN STREAM END),0) AS Streamed_On_Spotify
	FROM Spotify
	GROUP BY TRACK
	) AS T
	WHERE 
	Streamed_On_Spotify > Streamed_On_Youtube
	AND
	Streamed_On_Youtube <> 0;
   ```

### Advanced Level
1. Find the top 3 most-viewed tracks for each artist using window functions.
   ```sql
   WITH Ranking_Artist
	AS
	(SELECT
	ARTIST,
	TRACK,
	SUM(VIEWS) AS Total_Views,
	DENSE_RANK() OVER (PARTITION BY ARTIST ORDER BY ARTIST, SUM(VIEWS) DESC) AS DNS_RNK
	FROM Spotify
	GROUP BY ARTIST, TRACK
	ORDER BY ARTIST,Total_Views DESC
	)
	SELECT * FROM Ranking_Artist
	WHERE DNS_RNK <= 3;
   ```
2. Write a query to find tracks where the liveness score is above the average.
   ```sql
	SELECT
	TRACK,
	LIVENESS
	FROM Spotify
	WHERE LIVENESS > (SELECT AVG(LIVENESS) FROM Spotify);
   ```
3. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   
4. Find tracks where the energy-to-liveness ratio is greater than 1.2.
5. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.


Here’s an updated section for your **Spotify Advanced SQL Project and Query Optimization** README, focusing on the query optimization task you performed. You can include the specific screenshots and graphs as described.

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## How to Run the Project
1. Install PostgreSQL and pgAdmin (if not already installed).
2. Set up the database schema and tables using the provided normalization structure.
3. Insert the sample data into the respective tables.
4. Execute SQL queries to solve the listed problems.
5. Explore query optimization techniques for large datasets.

---

## Next Steps
- **Visualize the Data**: Use a data visualization tool like **Tableau** or **Power BI** to create dashboards based on the query results.
- **Expand Dataset**: Add more rows to the dataset for broader analysis and scalability testing.
- **Advanced Querying**: Dive deeper into query optimization and explore the performance of SQL queries on larger datasets.

---

## Contributing
If you would like to contribute to this project, feel free to fork the repository, submit pull requests, or raise issues.

---

## License
This project is licensed under the MIT License.
