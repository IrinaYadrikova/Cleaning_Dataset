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

## Create a table and upload the data

Here is an SQL query used for importing data from a CSV file into PostgreSQL:

```sql
CREATE TABLE real_estate_data (
    UniqueID SERIAL PRIMARY KEY,          -- Unique identifier for the record
    ParcelID VARCHAR(50),                 -- Parcel ID (usually a mix of numbers and letters)
    LandUse VARCHAR(100),                 -- Type of land use (e.g., "SINGLE FAMILY")
    PropertyAddress VARCHAR(255),         -- Address of the property
    SaleDate DATE,                        -- Date of the property sale
    SalePrice NUMERIC(15, 2),             -- Sale price of the property
    LegalReference VARCHAR(100),          -- Legal reference for the sale
    SoldAsVacant VARCHAR(10),             -- Whether the property was sold as vacant ("Yes"/"No")
    OwnerName VARCHAR(255),               -- Name of the property owner
    OwnerAddress VARCHAR(255),            -- Address of the property owner
    Acreage NUMERIC(10, 2),               -- Size of the property in acres
    TaxDistrict VARCHAR(100),             -- Tax district information
    LandValue NUMERIC(15, 2),             -- Value of the land
    BuildingValue NUMERIC(15, 2),         -- Value of the building on the land
    TotalValue NUMERIC(15, 2),            -- Total value of the property (Land + Building)
    YearBuilt INT,                        -- Year the building was built
    Bedrooms INT,                         -- Number of bedrooms
    FullBath INT,                         -- Number of full bathrooms
    HalfBath INT                          -- Number of half bathrooms
);

```
## Preview the dataset
```sql
select *
from real_estate_data
```
## Handle Missing Values
### Identify missing or NULL values:
```sql
select *
from real_estate_data
where propertyaddress is null
```
29 rows are NULL within the propertyaddress column.
--Explanation of the COALESCE Usage:
--COALESCE(a.propertyaddress, b.propertyaddress): This function will return the value from a.propertyaddress if it is not NULL. If a.propertyaddress is NULL, it will return the value from b.propertyaddress.

```sql
select 
	a.parcelid, 
	a.propertyaddress, 
	b.parcelid, 
	b.propertyaddress, 	
	COALESCE (a.propertyaddress,b.propertyaddress) AS final_propertyaddress
from real_estate_data a
join real_estate_data b on a.parcelid=b.parcelid
and a.uniqueid<>b.uniqueid
where a.propertyaddress is null;
```

### Update the data 
```sql
UPDATE real_estate_data
SET propertyaddress = (
	SELECT source.propertyaddress
	    FROM real_estate_data AS source
	    join real_estate_data on  source.parcelid = real_estate_data.parcelid
	    where  source.uniqueid <> real_estate_data.uniqueid
	      AND source.propertyaddress IS NOT NULL
		  LIMIT 1
		  )
	WHERE real_estate_data.propertyaddress IS NULL
```

### SET SalePrice = 0 or Parselid =0
```sql
SELECT * FROM real_estate_data
WHERE ParcelID IS NULL OR SalePrice IS NULL;
```
### Find duplicates. No duplicates were identifed:
```sql
SELECT uniqueid, COUNT(*)
FROM real_estate_data
GROUP BY uniqueid
HAVING COUNT(*) > 1;
```
### Eventhoght unique numner does not have a dublicates we need to check further
```sql
WITH ranked_data AS (
  SELECT 
    *,
    RANK() OVER (
      PARTITION BY 
        parcelid,
        propertyaddress,
        saleprice,
        legalreference
      ORDER BY uniqueid
    ) AS row_rank
  FROM real_estate_data
)
SELECT *
FROM ranked_data
WHERE row_rank > 1
```
--------------------------------
```sql
WITH ranked_data AS (
  SELECT 
    uniqueid,
    RANK() OVER (
      PARTITION BY 
        parcelid,
        propertyaddress,
        saleprice,
        legalreference
      ORDER BY uniqueid
    ) AS row_rank
  FROM real_estate_data
)
DELETE FROM real_estate_data
WHERE uniqueid IN (
  SELECT uniqueid
  FROM ranked_data
  WHERE row_rank > 1
);
```
### Breaking out Address into Idividual Columns (Address, City, nState)
```sql
select propertyaddress,
	SPLIT_PART(PropertyAddress, ',', 1) AS StreetAddress,	-- The part before the first comma
	 SPLIT_PART(PropertyAddress, ',', 2) AS City, 			-- The part between the first and second commas
	 Trim (SPLIT_PART(PropertyAddress, ',', 3)) AS State 	-- The part after the second comma
from real_estate_data;
```
```sql
ALTER TABLE real_estate_data
ADD COLUMN StreetAddress VARCHAR(255),
ADD COLUMN City VARCHAR(100),
ADD COLUMN State VARCHAR(10);
```
```sql
UPDATE real_estate_data
SET 
  StreetAddress = SPLIT_PART(PropertyAddress, ',', 1),
  City = SPLIT_PART(PropertyAddress, ',', 2),
  State = TRIM(SPLIT_PART(PropertyAddress, ',', 3));
```
### Change the column "soldasvacant" 
```sql
select distinct soldasvacant, count(*)
from real_estate_data
group by soldasvacant
```
### Delete unused columns 
```sql
select *
from real_estate_data
```
```sql
ALTER TABLE real_estate_data
DROP COLUMN taxdistrict;
```
### Standardize text columns (e.g., make all text lowercase or uppercase):
UPDATE real_estate_data
SET PropertyAddress = upper(PropertyAddress);

-------------
## Analyze the Data
### 1.Aggregate data
```sql
SELECT EXTRACT(YEAR FROM SaleDate) AS SaleYear, SUM(SalePrice) AS TotalSales
FROM real_estate_data
GROUP BY SaleYear
ORDER BY SaleYear;
```
### 2. Group and filter
Find the top 5 areas with the highest sales.

```sql
SELECT propertyaddress, saleprice, saledate
FROM real_estate_data
GROUP BY PropertyAddress, saleprice, saledate
ORDER BY saleprice DESC
LIMIT 5;
```
```sql
SELECT *
FROM real_estate_data
WHERE saleprice IN
(SELECT saleprice
FROM real_estate_data
--GROUP BY saleprice
--ORDER BY saleprice DESC
Limit 5) ;
```
```sql
WITH ranked_data AS (
  SELECT *,
    RANK() OVER (ORDER BY saleprice DESC) AS rank
  FROM real_estate_data
)
SELECT *
FROM ranked_data
WHERE rank =3;


