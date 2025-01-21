# Airbnb Analysis

### Project Overview

This repository contains an integrated dataset of Airbnb listings, focused primarily on Italian cities and neighborhoods. The goal of this project is to provide both raw data and analytical queries for anyone interested in exploring Airbnb trends, pricing, neighborhood diversity, host activities, and more.
Datasets Included
airbnb

Main listing-level data featuring details such as listing ID, host info, property details, price, and review scores.
airbnb_italian_city_grouped

Aggregated metrics grouped by city.
Fields include median prices, number of reviews, room type entropy, and other statistical summaries.
airbnb_italian_neighbourhood_grouped

Similar to airbnb_italian_city_grouped but aggregated at the neighborhood level.
Fields include neighborhood names, median prices, review scores, and additional aggregated metrics (e.g., host listing count median, accommodates median).
airbnb_italy_city_neighbourhoods

Contains a mapping of neighborhoods to broader groupings (e.g., neighbourhood_group) and city names.
Each dataset is joined by common fields (e.g., Neighbourhood, place) to facilitate more complex analyses and visualizations across multiple levels (city, neighborhood, listings, etc.).

### Data Source
This project’s data is sourced from Kaggle, an online community of data scientists and machine learning practitioners. Specifically, the Airbnb data used here was downloaded from the public Kaggle dataset:

Dataset Name: [Airbnb dataset](https://www.kaggle.com/datasets/heeraldedhia/airbnb-user-pathways).

### Tools

- Excel - Data cleaning [Download_here]{}
- Microsoft SQL server - Data analysis
- Power BI - Creating reports [Download_here]{}


### Data Cleaning\ Preparation

1- Importing Raw Data

We obtained the raw data from Kaggle’s Airbnb User Pathways dataset.
Files were loaded into our environment using SQL scripts to manage the data seamlessly.

2_ Merging Datasets

Multiple CSV files containing listing information, user activity, and location details were merged using common keys (e.g., user_id or listing_id).
We standardized naming conventions for columns to prevent conflicts and ensure consistent joins.

3_ Handling Missing Values

Missing data points (e.g., blank price fields, null coordinates) were identified 

4- Removing Duplicates

Duplicate rows (e.g., repeated user sessions) were identified using a combination of unique fields.
Duplicates were dropped or consolidated depending on relevance to the analysis.

5- Outlier Detection & Treatment

We used summary statistics (mean, median, standard deviation) to detect and potentially remove outliers (e.g., extremely high prices or anomalous number of reviews).
Depending on the analysis needs, certain outliers were truncated or kept if they provided meaningful insights.

6- Quality Checks

Data Type Consistency: Ensured numeric columns (prices, counts, ratings) were truly numeric.
Referential Integrity: Verified that references (e.g., listing_id in user logs) matched valid records in the main listing dataset.
Logical Constraints: Checked fields like beds_number or bathrooms_number for negative or unrealistic values.


### Exploratory Data Analysis

1- Which city has the highest average listing price, and how does it compare across different periods?

2- Do neighborhoods with higher median prices also exhibit higher median review scores, or is there a negative correlation?

3- Which neighborhood shows the greatest variation in prices (based on median absolute deviation), and why might that be?

4- How does the number of reviews per month (median) differ between cities, and what factors might influence these differences?

5- What is the relationship between host_total_listings_count_median and review_scores_rating_median?

6- Which neighborhoods have the most diverse room types, as indicated by higher room_type_shannon_entropy values?

7- Are there notable seasonal trends in prices or reviews when comparing different period values?

8- Which cities or neighborhoods are outliers, either in extremely high or low price_median or number_of_reviews_ltm_median?

9- Does a higher accommodates_median correlate with increased listing prices across different neighborhoods?

10- Which neighborhoods or cities have the widest range of review_scores_* metrics, indicating inconsistency in guest experiences?

### Data Analysis

-- Retrieve listings
```sql
SELECT 
    [Listings_id], [Last_year_reviews], [Host_since], [Host_is_superhost], 
    [Host_number_of_listings], [Neighbourhood], [Beds_number], 
    [Bedrooms_number], [Property_type], [Maximum_allowed_guests], 
    [Price], [Total_reviews], [Rating_score], [Accuracy_score], 
    [Cleanliness_score], [Checkin_score], [Communication_score], 
    [Location_score], [Value_for_money_score], [Reviews_per_month], 
    [City], [Season], [Bathrooms_number], [Bathrooms_type], 
    [Coordinates], [Date_of_scraping]
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Total listings count
```sql
SELECT COUNT(*) AS TotalListings 
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Distinct cities
```sql
SELECT DISTINCT [City] 
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Inspect missing data
```sql
SELECT * 
FROM [Abiodun airbnb].[dbo].[airbnb]
WHERE [Price] IS NULL OR [Rating_score] IS NULL;
```
-- Top-rated properties
```sql
SELECT * 
FROM [Abiodun airbnb].[dbo].[airbnb]
WHERE [Rating_score] >= 4.9;
```
-- Affordable listings in Centro Storico
```sql
SELECT * 
FROM [Abiodun airbnb].[dbo].[airbnb]
WHERE [Neighbourhood] = 'Centro Storico' AND [Price] < 100;
```
-- Average price and total listings by property type
```sql
SELECT [Property_type], 
       AVG([Price]) AS AvgPrice, 
       COUNT([Listings_id]) AS TotalListings
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [Property_type];
```
-- Reviews analysis by city
```sql
SELECT [City], 
       SUM([Total_reviews]) AS TotalReviews, 
       AVG([Reviews_per_month]) AS AvgReviewsPerMonth
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [City];
```
-- Listings by host year
```sql
SELECT YEAR([Host_since]) AS HostYear, 
       COUNT([Listings_id]) AS TotalListings
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY YEAR([Host_since])
ORDER BY HostYear;
```
-- Seasonal review trends
```sql
SELECT [Season], 
       AVG([Reviews_per_month]) AS AvgMonthlyReviews
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [Season]
ORDER BY AvgMonthlyReviews DESC;
```
-- Top 10 most expensive listings
```sql
SELECT TOP 10 [Listings_id], [Price], [City], [Neighbourhood]
FROM [Abiodun airbnb].[dbo].[airbnb]
ORDER BY [Price] DESC;
```
-- Rank listings by ratings within a city
```sql
SELECT [City], [Listings_id], [Rating_score],
       RANK() OVER (PARTITION BY [City] ORDER BY [Rating_score] DESC) AS Rank
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Revenue potential per listing
```sql
SELECT [Listings_id], 
       [Price] * [Maximum_allowed_guests] AS RevenuePotential
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Normalized rating score
```sql
SELECT [Listings_id], 
       ([Rating_score] * [Total_reviews]) / 100 AS NormalizedScore
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Average price and rating by neighbourhood
```sql
SELECT [Neighbourhood], 
       AVG([Price]) AS AvgPrice, 
       AVG([Rating_score]) AS AvgRating
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [Neighbourhood];
```
-- Superhost performance
```sql
SELECT [Host_is_superhost], 
       AVG([Price]) AS AvgPrice, 
       AVG([Rating_score]) AS AvgRating, 
       AVG([Total_reviews]) AS AvgReviews
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [Host_is_superhost];
```
-- Listings within a specific coordinate range
```sql
SELECT * 
FROM [Abiodun airbnb].[dbo].[airbnb]
WHERE [Coordinates] LIKE '40.7%, -73.9%';
```
-- Cluster analysis by city
```sql
SELECT [City], 
       COUNT([Listings_id]) AS TotalListings
FROM [Abiodun airbnb].[dbo].[airbnb]
GROUP BY [City]
ORDER BY TotalListings DESC;
```
-- Outliers in pricing
```sql
SELECT *,
       CASE 
           WHEN [Price] > AVG([Price]) OVER (PARTITION BY [City]) * 1.5 THEN 'High Outlier'
           WHEN [Price] < AVG([Price]) OVER (PARTITION BY [City]) * 0.5 THEN 'Low Outlier'
           ELSE 'Normal'
       END AS PriceOutlierStatus
FROM [Abiodun airbnb].[dbo].[airbnb];
```
-- Listings with multiple bedrooms
```sql
SELECT [Listings_id], [Neighbourhood], [Price], [Bedrooms_number], 
       [Maximum_allowed_guests], [Rating_score]
FROM [Abiodun airbnb].[dbo].[airbnb]
WHERE [Bedrooms_number] > 2
ORDER BY [Price] DESC;
```
-- Listings by Neighbourhood Group and Property Type
```sql
SELECT 
    c.[neighbourhood_group], 
    a.[Property_type], 
    COUNT(a.[Listings_id]) AS TotalListings
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group], a.[Property_type]
ORDER BY 
    c.[neighbourhood_group], TotalListings DESC;
```
-- Identify Neighbourhood Groups with the Most Superhosts
```sql
SELECT 
    c.[neighbourhood_group], 
    COUNT(a.[Listings_id]) AS SuperhostListings
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
WHERE 
    a.[Host_is_superhost] = 't'
GROUP BY 
    c.[neighbourhood_group]
ORDER BY 
    SuperhostListings DESC;
```
-- Median Price Deviation in Each Neighbourhood Group
```sql
SELECT 
    c.[neighbourhood_group], 
    b.[price_median], 
    b.[price_median_abs_deviation]
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] b
ON 
    a.[Neighbourhood] = b.[neighbourhood_mode]
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group], b.[price_median], b.[price_median_abs_deviation]
ORDER BY 
    b.[price_median] DESC;
```
-- Listings with the Best Location Scores
```sql
SELECT 
    a.[Listings_id], 
    a.[Location_score], 
    c.[neighbourhood_group], 
    a.[Price]
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
WHERE 
    a.[Location_score] >= 9
ORDER BY 
    a.[Location_score] DESC, a.[Price] ASC;
```
-- Host Activity Across Neighbourhood Groups
```sql
SELECT 
    c.[neighbourhood_group], 
    AVG(a.[Host_number_of_listings]) AS AvgListingsPerHost, 
    COUNT(DISTINCT a.[Listings_id]) AS UniqueListings
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group]
ORDER BY 
    AvgListingsPerHost DESC, UniqueListings DESC;
```
-- High Demand Areas: Listings with High Review Frequencies
```sql
SELECT 
    a.[Listings_id], 
    a.[Reviews_per_month], 
    c.[neighbourhood_group]
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
WHERE 
    a.[Reviews_per_month] > 3
ORDER BY 
    a.[Reviews_per_month] DESC;
```
-- Metrics for Cleanliness Score, Check-in Score, Location Score, Value of Money Score
```sql
SELECT 
    c.[neighbourhood_group],
    AVG(a.[Cleanliness_score]) AS AvgCleanlinessScore,
    AVG(a.[Checkin_score]) AS AvgCheckinScore,
    AVG(a.[Location_score]) AS AvgLocationScore,
    AVG(a.[Value_for_money_score]) AS AvgValueForMoneyScore,
    AVG(a.[Reviews_per_month]) AS AvgReviewsPerMonth
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group]
ORDER BY 
    c.[neighbourhood_group];
```
-- Listings with the Highest Review Per Month
```sql
SELECT 
    a.[Listings_id],
    c.[neighbourhood_group],
    a.[Cleanliness_score],
    a.[Checkin_score],
    a.[Location_score],
    a.[Value_for_money_score],
    a.[Reviews_per_month]
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
WHERE 
    a.[Reviews_per_month] IS NOT NULL
ORDER BY 
    a.[Reviews_per_month] DESC;
```
-- Approximate Median Scores
```sql
WITH RankedScores AS (
    SELECT 
        c.[neighbourhood_group],
        a.[Cleanliness_score],
        a.[Checkin_score],
        a.[Location_score],
        a.[Value_for_money_score],
        a.[Reviews_per_month],
        ROW_NUMBER() OVER (PARTITION BY c.[neighbourhood_group] ORDER BY a.[Cleanliness_score]) AS CleanlinessRank,
        ROW_NUMBER() OVER (PARTITION BY c.[neighbourhood_group] ORDER BY a.[Checkin_score]) AS CheckinRank,
        ROW_NUMBER() OVER (PARTITION BY c.[neighbourhood_group] ORDER BY a.[Location_score]) AS LocationRank,
        ROW_NUMBER() OVER (PARTITION BY c.[neighbourhood_group] ORDER BY a.[Value_for_money_score]) AS ValueRank,
        ROW_NUMBER() OVER (PARTITION BY c.[neighbourhood_group] ORDER BY a.[Reviews_per_month]) AS ReviewsRank,
        COUNT(*) OVER (PARTITION BY c.[neighbourhood_group]) AS TotalListings
    FROM 
        [Abiodun airbnb].[dbo].[airbnb] a
    LEFT JOIN 
        [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
    ON 
        a.[Neighbourhood] = c.[neighbourhood]
)
SELECT 
    neighbourhood_group,
    AVG(CASE WHEN CleanlinessRank = (TotalListings + 1) / 2 THEN [Cleanliness_score] END) AS MedianCleanlinessScore,
    AVG(CASE WHEN CheckinRank = (TotalListings + 1) / 2 THEN [Checkin_score] END) AS MedianCheckinScore,
    AVG(CASE WHEN LocationRank = (TotalListings + 1) / 2 THEN [Location_score] END) AS MedianLocationScore,
    AVG(CASE WHEN ValueRank = (TotalListings + 1) / 2 THEN [Value_for_money_score] END) AS MedianValueForMoneyScore,
    AVG(CASE WHEN ReviewsRank = (TotalListings + 1) / 2 THEN [Reviews_per_month] END) AS MedianReviewsPerMonth
FROM 
    RankedScores
GROUP BY 
    neighbourhood_group
ORDER BY 
    neighbourhood_group;
	 
	 SELECT 
    COUNT(*) AS TotalRows,
    SUM(CASE WHEN Cleanliness_score IS NULL THEN 1 ELSE 0 END) AS MissingCleanlinessScore,
    SUM(CASE WHEN Location_score IS NULL THEN 1 ELSE 0 END) AS MissingLocationScore,
    SUM(CASE WHEN Checkin_score IS NULL THEN 1 ELSE 0 END) AS MissingCheckinScore,
    SUM(CASE WHEN Value_for_money_score IS NULL THEN 1 ELSE 0 END) AS MissingValueScore,
    SUM(CASE WHEN Reviews_per_month IS NULL THEN 1 ELSE 0 END) AS MissingReviews
FROM 
    [Abiodun airbnb].[dbo].[airbnb];

	SELECT 
    [Listings_id], 
    COUNT(*) AS DuplicateCount
FROM 
    [Abiodun airbnb].[dbo].[airbnb]
GROUP BY 
    [Listings_id]
HAVING 
    COUNT(*) > 1;

	SELECT 
    MIN([Price]) AS MinPrice,
    MAX([Price]) AS MaxPrice,
    MIN([Reviews_per_month]) AS MinReviews,
    MAX([Reviews_per_month]) AS MaxReviews
FROM 
    [Abiodun airbnb].[dbo].[airbnb];

	SELECT 
    AVG([Cleanliness_score]) AS AvgCleanlinessScore,
    AVG([Checkin_score]) AS AvgCheckinScore,
    AVG([Location_score]) AS AvgLocationScore,
    AVG([Value_for_money_score]) AS AvgValueForMoneyScore,
    AVG([Price]) AS AvgPrice,
    AVG([Reviews_per_month]) AS AvgReviews
FROM 
    [Abiodun airbnb].[dbo].[airbnb];

	WITH Stats AS (
    SELECT 
        AVG(Price) AS AvgPrice,
        AVG(Location_score) AS AvgLocationScore,
        STDEV(Price) AS StdDevPrice,
        STDEV(Location_score) AS StdDevLocationScore
    FROM 
        [Abiodun airbnb].[dbo].[airbnb]
),
Deviation AS (
    SELECT 
        Price,
        Location_score,
        (Price - Stats.AvgPrice) AS PriceDeviation,
        (Location_score - Stats.AvgLocationScore) AS LocationDeviation
    FROM 
        [Abiodun airbnb].[dbo].[airbnb], Stats
),
Covariance AS (
    SELECT 
        SUM(PriceDeviation * LocationDeviation) AS Covariance,
        COUNT(*) AS N
    FROM 
        Deviation
)
SELECT 
    (Covariance.Covariance / Covariance.N) / 
    (Stats.StdDevPrice * Stats.StdDevLocationScore) AS CorrelationCoefficient
FROM 
    Covariance, Stats;

	SELECT 
    c.[neighbourhood_group],
    AVG(a.[Cleanliness_score]) AS AvgCleanlinessScore,
    AVG(a.[Location_score]) AS AvgLocationScore,
    AVG(a.[Value_for_money_score]) AS AvgValueForMoneyScore
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group]
ORDER BY 
    c.[neighbourhood_group];

	SELECT 
    c.[neighbourhood_group], 
    AVG(a.[Reviews_per_month]) AS AvgReviewsPerMonth,
    AVG(a.[Price]) AS AvgPrice
FROM 
    [Abiodun airbnb].[dbo].[airbnb] a
LEFT JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italy_city_neighbourhoods] c
ON 
    a.[Neighbourhood] = c.[neighbourhood]
GROUP BY 
    c.[neighbourhood_group]
ORDER BY 
    AvgReviewsPerMonth DESC;
```
-- Compare neighborhoods within cities for average prices and ratings
```sql
SELECT 
    C.[place] AS city, 
    N.[neighbourhood], 
    AVG(N.[price_median]) AS avg_neighbourhood_price, 
    AVG(N.[review_scores_rating_median]) AS avg_neighbourhood_rating,
    AVG(C.[price_median]) AS avg_city_price, 
    AVG(C.[review_scores_rating_median]) AS avg_city_rating
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place]
GROUP BY 
    C.[place], N.[neighbourhood]
ORDER BY 
    avg_neighbourhood_price DESC, avg_neighbourhood_rating DESC;
```
-- Analyze host diversity across cities and neighborhoods
```sql
SELECT 
    C.[place] AS city, 
    N.[neighbourhood], 
    AVG(N.[host_total_listings_count_median]) AS avg_neighbourhood_hosts, 
    AVG(C.[host_total_listings_count_median]) AS avg_city_hosts,
    STDEV(N.[host_total_listings_count_median]) AS host_std_dev
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place]
GROUP BY 
    C.[place], N.[neighbourhood]
ORDER BY 
    avg_neighbourhood_hosts DESC;
```
-- Neighborhoods with the most diversity in room types
```sql
SELECT 
    N.[neighbourhood], 
    C.[place] AS city, 
    N.[room_type_mode], 
    N.[room_type_shannon_entropy] AS diversity_score
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
ON 
    N.[place] = C.[place]
WHERE 
    N.[room_type_shannon_entropy] > (
        SELECT AVG([room_type_shannon_entropy]) 
        FROM [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped]
    )
ORDER BY 
    diversity_score DESC;
```
-- Compare price variation between cities and neighborhoods
```sql
SELECT 
    C.[place] AS city, 
    N.[neighbourhood], 
    C.[price_median] AS city_price_median, 
    N.[price_median] AS neighbourhood_price_median, 
    ABS(C.[price_median] - N.[price_median]) AS price_difference
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place]
ORDER BY 
    price_difference DESC;
```
-- Analyze price trends over time by city and neighborhood
```sql
SELECT 
    C.[place] AS city, 
    N.[neighbourhood], 
    N.[period], 
    AVG(N.[price_median]) AS avg_price, 
    AVG(N.[reviews_per_month_median]) AS avg_reviews_per_month
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place]
GROUP BY 
    C.[place], N.[neighbourhood], N.[period]
ORDER BY 
    N.[period], avg_price DESC;
```
-- Compare geographic differences using latitude and longitude
```sql
SELECT 
    N.[neighbourhood], 
    C.[place] AS city, 
    N.[latitude_median], 
    N.[longitude_median], 
    N.[price_median], 
    N.[review_scores_rating_median]
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place]
WHERE 
    N.[latitude_median] > 43.0 AND N.[longitude_median] < 12.0
ORDER BY 
    N.[price_median] DESC;
```
-- Predict host activity using reviews per month
```sql
SELECT 
    C.[place] AS city, 
    N.[neighbourhood], 
    N.[reviews_per_month_median], 
    N.[host_total_listings_count_median],
    (N.[reviews_per_month_median] * 0.8) + 10 AS predicted_host_count
FROM 
    [Abiodun airbnb].[dbo].[airbnb_italian_city_grouped] C
JOIN 
    [Abiodun airbnb].[dbo].[airbnb_italian_neighbourhood_grouped] N
ON 
    C.[place] = N.[place];
```
### Result/ Findings
Overall Listing Insights

Total Listings: The dataset contains several thousand Airbnb listings, covering a wide range of cities and neighborhoods across Italy.
Average Price: The median nightly price was moderately high in major urban centers (e.g., Milan, Rome), while smaller towns tended to have lower average prices.
City-Level Analysis

Highest Median Price: Milan consistently showed the highest price_median in the airbnb_italian_city_grouped table, suggesting stronger demand or a more premium market.
Review Trends: Cities like Florence and Venice exhibited comparatively higher monthly reviews, indicating robust tourism traffic.
Neighborhood-Level Analysis

Price Variation: Certain neighborhoods in Rome and Milan had significantly larger price deviations (based on price_median_abs_deviation), indicating a mix of both budget and luxury listings in the same areas.
Diversity of Room Types: Neighborhoods with high room_type_shannon_entropy (e.g., central historic districts) showed a wider mix of entire homes, private rooms, and shared rooms.
Host Activity

Superhosts vs. Non-Superhosts: Queries revealed that superhosts generally maintain higher review_scores_rating_median and more consistent occupancy rates.
Host Listings: Neighborhoods characterized by professional or commercial hosts (high host_total_listings_count_median) often had standardized pricing but slightly lower rating scores, possibly due to a more commercial experience.
Reviews and Ratings

Correlation with Price: A modest positive relationship between price_median and review_scores_rating_median was observed, suggesting that pricier listings often maintain slightly higher ratings, though exceptions exist.
Review Volume: The reviews_per_month_median indicator was higher for coastal or tourist-centric neighborhoods, reflecting strong seasonal demand.
Geospatial Findings

Latitude/Longitude Clusters: By examining latitude_median and longitude_median, popular tourist areas (e.g., city centers near major landmarks) clustered with higher prices and review counts.
Outliers: A few neighborhoods well outside central tourist zones displayed unexpectedly high prices, possibly indicating luxury villas or unique listings.
Seasonality

Time-Based Patterns: Aggregating data by period showed clear peaks in price_median and reviews_per_month_median during summer months, correlating with vacation season in many Italian destinations.
Potential Opportunities

Price Optimization: Hosts could consider dynamic pricing strategies in neighborhoods with large deviations in listing costs to remain competitive.
Rating Improvement: Some neighborhoods with lower review_scores_rating_median might benefit from improved amenities or host responsiveness to attract more positive reviews.
Data Quality Observations

Missing Data: A small percentage of listings had incomplete information (e.g., missing coordinates or bathrooms). These were either imputed or excluded from certain analyses.
Outliers: Extremely high-priced or unusually low-rated listings appeared in certain queries, warranting further investigation or potential data cleansing.

