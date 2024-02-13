# Cleaning data in SQL Queries

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning](#data-cleaning)
- [Summary](#summary)


### Project Overview
This project shows the process of cleaning data in SQL to eliminate errors and inaccuracies for effective data analysis.

### Data Sources
The primary dataset used in this project is the "Nashville_Housing_CSV", containing detailed information about housing in Nashville.

### Tools
- SQL Server

### Data Cleaning
I Performed the following tasks during the cleaning process
1. Renaming table
2. Populate Property Address
3. Breaking out Address into individual columns (Address, City, State)
4. Handling missing Values 
5. Removing Duplicates


```SQL
--Renaming table to a shorter name 
EXEC sp_rename 'Nashville Housing Data for Data Cleaning - Sheet1', 'NashvilleHousing';

SELECT * 
FROM dbo.NashvilleHousing 


--Populate property Address data
SELECT * 
FROM dbo.NashvilleHousing 
WHERE PropertyAddress is NULL
ORDER BY ParcelID


SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM dbo.NashvilleHousing a
JOIN dbo.NashvilleHousing b 
  ON a.ParcelID = b.ParcelID
  AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress is null 

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM dbo.NashvilleHousing a
JOIN dbo.NashvilleHousing b 
  ON a.ParcelID = b.ParcelID
  AND a.[UniqueID] <> b.[UniqueID]
  WHERE a.PropertyAddress is null 


--Breaking out Address into individual columns (Address, City, State)

    --Using substrings to break PropertyAddress into Address and City

SELECT PropertyAddress
FROM dbo.NashvilleHousing 
--WHERE PropertyAddress is NULL
--ORDER BY ParcelID

SELECT
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address
,SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) as Address
FROM dbo.NashvilleHousing 

ALTER TABLE NashvilleHousing 
ADD PropertySplitAddress NVARCHAR(255)

UPDATE NashvilleHousing 
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)


ALTER TABLE NashvilleHousing 
ADD PropertySplitCity NVARCHAR(255)

UPDATE NashvilleHousing 
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))



   --Using PARSENAME to break OwnerAddress into Address, City and State

SELECT 
PARSENAME(REPLACE(OwnerAddress, ',','.'),3)
,PARSENAME(REPLACE(OwnerAddress, ',','.'),2)
,PARSENAME(REPLACE(OwnerAddress, ',','.'),1)
FROM dbo.NashvilleHousing

ALTER TABLE NashvilleHousing 
ADD OwnerSplitAddress NVARCHAR(255)

UPDATE NashvilleHousing 
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',','.'),3)

ALTER TABLE NashvilleHousing 
ADD OwnerSplitCity NVARCHAR(255)

UPDATE NashvilleHousing 
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',','.'),2)

ALTER TABLE NashvilleHousing 
ADD OwnerSplitState NVARCHAR(255)

UPDATE NashvilleHousing 
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',','.'),1)


--Change Y AND n to Yes and No in "Sold as Vacant" field

SELECT DISTINCT(SoldAsVacant),COUNT(SoldAsVacant)
FROM dbo.NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2

SELECT SoldAsVacant
, CASE WHEN SoldAsVacant ='Y' THEN 'YES'
       WHEN SoldAsVacant ='N' THEN 'NO'
       ELSE SoldAsVacant
       END
FROM dbo.NashvilleHousing

UPDATE NashvilleHousing
SET SoldASVacant = CASE WHEN SoldAsVacant ='Y' THEN 'YES'
       WHEN SoldAsVacant ='N' THEN 'NO'
       ELSE SoldAsVacant
       END



--Removing Duplicates

WITH RowNumCTE AS(
SELECT *
  ,ROW_NUMBER() OVER (
    PARTITION BY ParcelID,
                 PropertyAddress,
                 SalePrice,
                 SaleDate,
                 LegalReference
                 ORDER BY
                   UniqueID
                  ) row_num
FROM dbo.NashvilleHousing
--ORDER BY ParcelID
)


DELETE
FROM RowNumCTE
WHERE row_num > 1
--ORDER BY PropertyAddress


SELECT *
FROM RowNumCTE
WHERE row_num > 1
ORDER BY PropertyAddress



--Delete Unused Columns
SELECT *
FROM dbo.NashvilleHousing

ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress
```

### Summary 
In this project, I tried some of the common data-cleaning techniques for tidy data. Data should be complete, correct, and relevant for effective analysis in other to provide insight into the problem you're trying to solve.



