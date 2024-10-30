# Data Cleaning

##Covid_deaths data set,data-cleaning

This data analysis project aims to provide insights in to the cleaning of coviddeaths dataset 

###select *
from PortfoliProject.dbo.NashvilleHousing 

-------------------------------------------------------------------------------------------------------------------

--Standardize Date Format

select SaleDateConverted, CONVERT(Date,SaleDate)
from PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)]


update [Nashville Housing Data for Data Cleaning (reuploaded)]
SET SaleDate = CONVERT(Date,SaleDate)


ALTER TABLE [Nashville Housing Data for Data Cleaning (reuploaded)]
Add SaleDateConverted Date;

update [Nashville Housing Data for Data Cleaning (reuploaded)]
SET SaleDateConverted = CONVERT(Date,SaleDate)

--Populate Property Address data

select *
from PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)]
--where PropertyAddress is null
order by ParcelID

select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
from PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)] a
JOIN PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)] b
     on a.ParcelID = b.ParcelID
	 AND a.[UniqueID] <> b.[UniqueID]
where a.PropertyAddress is null

update a
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
from PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)] a
JOIN PortfoliProject.dbo.[Nashville Housing Data for Data Cleaning (reuploaded)] b
     on a.ParcelID = b.ParcelID
	 AND a.[UniqueID] <> b.[UniqueID]
where a.PropertyAddress is null


------------------------------------------------------------------------------------------------------------------------------------

--Breaking out Address into Indivisual Columns (Address, City, State)



select PropertyAddress
from PortfoliProject.dbo.NashvilleHousing
--where PropertyAddress is null
--order by ParcelID

SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address
,SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) as Address

from PortfoliProject.dbo.NashvilleHousing


ALTER TABLE PortfoliProject.dbo.NashvilleHousing
Add PropertySplitAdddress Nvarchar(255);


update PortfoliProject.dbo.NashvilleHousing
SET PropertySplitAdddress =  SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) ;


ALTER TABLE PortfoliProject.dbo.NashvilleHousing
Add PropertySplitCity Nvarchar(255);

update PortfoliProject.dbo.NashvilleHousing
SET  PropertySplitCity = SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) 

SELECT *
FROM PortfoliProject.dbo.NashvilleHousing


SELECT OwnerAddress
from PortfoliProject.dbo.NashvilleHousing

select
PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)
,PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2)
,PARSENAME(REPLACE(OwnerAddress, ',', '.') ,  1)
from PortfoliProject.dbo.NashvilleHousing


ALTER TABLE PortfoliProject.dbo.NashvilleHousing
Add OwnerSplitAdddress Nvarchar(255);


update PortfoliProject.dbo.NashvilleHousing
SET OwnerSplitAdddress =  PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)


ALTER TABLE PortfoliProject.dbo.NashvilleHousing
Add OwnerSplitCity Nvarchar(255);

update PortfoliProject.dbo.NashvilleHousing
SET  OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2) 

ALTER TABLE PortfoliProject.dbo.NashvilleHousing
Add OwnerSplitState Nvarchar(255);

update PortfoliProject.dbo.NashvilleHousing
SET  OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.') ,  1)

SELECT *
from PortfoliProject.dbo.NashvilleHousing

---------------------------------------------------------------------------------------------------------------------------------------------

-- change Y and N to yes and No in 'Sold as Vacant' field

select Distinct(SoldAsVacant), Count(SoldAsVacant)
From PortfoliProject.dbo.NashvilleHousing
Group by SoldAsVacant
order by 2

select SoldAsVacant
, CASE When SoldAsVacant = 1 THEN 'Yes'
       When SoldAsVacant = 0 THEN 'No' 
	   ELSE SoldAsVacant
	   END
From PortfoliProject.dbo.NashvilleHousing

------------------------------------------------------------------------------------------------------------------------------------------

--Remove Duplicate

WITH RowNumCTE AS(
Select *,
   ROW_NUMBER() OVER (
   PARTITION BY ParcelID,
               PropertyAddress,
			   SalePrice,
			   SaleDate,
			   LegalReference
			   ORDER BY
			      UniqueID
				  ) row_num

From PortfoliProject.dbo.NashvilleHousing
--order by ParcelID
)
SELECT *
from RowNumCTE
Where row_num > 1
order by  PropertyAddress
 


---------------------------------------------------------------------------------------------

--Delete Unused Columns


SELECT *
from PortfoliProject.dbo.NashvilleHousing


ALTER TABLE PortfoliProject.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress

ALTER TABLE PortfoliProject.dbo.NashvilleHousing
DROP COLUMN SaleDate
