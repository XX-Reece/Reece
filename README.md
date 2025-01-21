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

Missing data points (e.g., blank price fields, null coordinates) were identified using Pandas’ isnull() function and SQL WHERE ... IS NULL queries.
Based on the nature of each field, we either:
Filled them with median or mean values (for numerical fields),
Imputed them with placeholders (for categorical fields), or
Excluded rows with excessive null values if they were unfit for analysis.

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

