# Cleaning_Dataset and Data Analysis Demonstration Project

This project is created for demonstration purposes, to showcase my ability to work with data and perform SQL queries for data cleaning.

## Purpose
The main objective of this project is to demonstrate how I can efficiently clean and analyse data using SQL. Throughout the project, I focus on various data-cleaning techniques, ensuring the dataset is prepared for further analysis or reporting.

## YouTube Channel

For further resources and tutorials, please check out Alex the Analyst's YouTube channel. He provides great content on data analysis, SQL, and more!

- [Alex The Analyst - YouTube Channel](https://www.youtube.com/c/AlexTheAnalyst)

## Project Highlights

- **Data Cleaning**: I applied multiple SQL queries to clean and normalize the dataset. This includes handling missing values, standardizing data formats, and removing duplicates.
- **SQL Skills**: Demonstrated proficiency in writing complex SQL queries, including joins, aggregations, and subqueries, to clean and prepare data for analysis.

## Tools Used

- **SQL**: For data extraction, transformation, and cleaning.
- **PostgreSQL**: As the database management system.
- **PGAdmin**: This is for managing and querying the database.

## Creat a table and upload the data

Here is an SQL query used for importing data from a CSV file into PostgreSQL:

```sql
INSERT INTO real_estate_data (ParcelID, LandUse, PropertyAddress, SaleDate, SalePrice, LegalReference, SoldAsVacant, OwnerName, OwnerAddress, Acreage, TaxDistrict, LandValue, BuildingValue, TotalValue, YearBuilt, Bedrooms, FullBath, HalfBath)
SELECT
    ParcelID,
    LandUse,
    PropertyAddress,
    TO_DATE(SaleDate, 'YYYY-MM-DD'),
    CAST(SalePrice AS NUMERIC),
    LegalReference,
    SoldAsVacant,
    OwnerName,
    OwnerAddress,
    CASE WHEN Acreage = '' THEN NULL ELSE CAST(Acreage AS NUMERIC) END,
    TaxDistrict,
    CASE WHEN LandValue = '' THEN NULL ELSE CAST(LandValue AS NUMERIC) END,
    CASE WHEN BuildingValue = '' THEN NULL ELSE CAST(BuildingValue AS NUMERIC) END,
    CASE WHEN TotalValue = '' THEN NULL ELSE CAST(TotalValue AS NUMERIC) END,
    CAST(YearBuilt AS INT),
    CAST(Bedrooms AS INT),
    CAST(FullBath AS INT),
    CAST(HalfBath AS INT)
FROM staging_real_estate_data;
```



