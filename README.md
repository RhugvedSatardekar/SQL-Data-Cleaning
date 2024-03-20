```sql
-- SQL Data Cleaning/Transformation Project Summary

## Overview:
This project involves cleaning and transforming data from the NashvilleHousing dataset, which was imported into SQL Server Management Studio (SSMS) from an Excel file. The tasks include standardizing date formats, populating missing property addresses, splitting address columns, replacing values, removing duplicates, and dropping unnecessary columns to prepare the dataset for analysis and reporting.

## Steps:

1. **Import Data from Excel into SSMS:**
   - The NashvilleHousing dataset was imported into SSMS from an Excel file for data cleaning and transformation.

2. **Standardize Date Format:**
   - Added a new column SaleDateConverted and updated it with standardized date formats from the SaleDate column.

```sql
select top (100) * from NashvilleHousing

Alter table NashvilleHousing
add SaleDateConverted Date;

Update NashvilleHousing
set SaleDateConverted = Convert(Date, SaleDate)

select SaleDateConverted, Convert(Date, SaleDate) SaleDate2 from DataCleaningProject..NashvilleHousing
```

3. **Populate Property Address Data:**
   - Identified and populated missing property addresses based on matching ParcelID values.

```sql
select a.[UniqueID ],a.ParcelID, a.PropertyAddress,ISNULL(a.PropertyAddress,b.PropertyAddress) as PrpertyAddressA
from DataCleaningProject..NashvilleHousing as a
join DataCleaningProject..NashvilleHousing as b
	on a.ParcelID = b.ParcelID
	and 
	a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

Update a
Set PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
from DataCleaningProject..NashvilleHousing as a
join DataCleaningProject..NashvilleHousing as b
	on a.ParcelID = b.ParcelID
	and 
	a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

Select * from NashvilleHousing
order by ParcelID
```

4. **Split PropertyAddress into Separate Columns (Address, City):**
   - Split the PropertyAddress column into Address and City columns using string functions.

```sql
-- Splitting PropertyAddress into seperate columns (Address, City)

Select PropertyAddress from NashvilleHousing

Select 
 SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1) as address,
 SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, len(PropertyAddress)) as City
from NashvilleHousing
```

5. **Added Columns to Table Schema:**
   - Added Address and City columns to the NashvilleHousing table.

```sql
-- Added Columns to Table Schema
Alter table NashvilleHousing
add Address nvarchar(256), City nvarchar(256), State nvarchar(256);
```

6. **Populated Values into Newly Added Columns:**
   - Updated the Address and City columns with values extracted from the PropertyAddress column.

```sql
-- Populated Values into newly added columns

Update NashvilleHousing
set Address = SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1),
City = SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, len(PropertyAddress))

Select PropertyAddress, Address, City
from NashvilleHousing
```

7. **Split OwnerAddress into Separate Columns (Address, City, State):**
   - Split the OwnerAddress column into Address, City, and State columns using PARSENAME function.

```sql
-- Splitting OwnerAddress into seperate columns (Address, City, State)

Select OwnerAddress from NashvilleHousing

select 
PARSENAME(replace(OwnerAddress, ',', '.'),3) as Address,
PARSENAME(replace(OwnerAddress, ',', '.'),2) as City,
PARSENAME(replace(OwnerAddress, ',', '.'),1) as State
from NashvilleHousing

Alter table NashvilleHousing
add OwnerNewAddress nvarchar(256), OwnerCity nvarchar(256), OwnerState nvarchar(256);

Update NashvilleHousing
set OwnerNewAddress = PARSENAME(replace(OwnerAddress, ',', '.'),3),
OwnerCity = PARSENAME(replace(OwnerAddress, ',', '.'),2) ,
OwnerState = PARSENAME(replace(OwnerAddress, ',', '.'),1)

Select OwnerAddress, OwnerNewAddress, OwnerCity, OwnerState
from NashvilleHousing
```

8. **Replace Y and N with Yes and No in SoldAsVacant Column:**
   - Created a new column SoldAsVacantNew to replace 'Y' and 'N' values with 'Yes' and 'No' respectively.

```sql
-- Replacing Y and N to Yes and No in "Sold as vacand Column

select * from NashvilleHousing

select Distinct(SoldAsVacant), Count(SoldAsVacant)
from NashvilleHousing
group by SoldAsVacant
Order by 2


select SoldAsVacant,
	case 
		when SoldAsVacant = 'Y' then 'Yes'
		when SoldAsVacant = 'N' then 'No'
		else SoldAsVacant
	end as SoldAsVacentNew
from NashvilleHousing

Alter table NashvilleHousing
add SoldAsVacantNew nvarchar(256);

Update NashvilleHousing
set SoldAsVacantNew = case 
		when SoldAsVacant = 'Y' then 'Yes'
		when SoldAsVacant = 'N' then 'No'
		else SoldAsVacant
	end

select distinct(SoldAsVacant), count(SoldAsVacant), SoldAsVacantNew, count(SoldAsVacantNew)
from NashvilleHousing
group by SoldAsVacant, SoldAsVacantNew
order by 2,4	
```

9. **Remove Duplicates:**
   - Identified and removed duplicate rows based on specified criteria using a common table expression (CTE).

```sql
-- Remove Duplicates

with RowNumCTE as(
select *,
	row_number() over (
	partition by ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 
	Order by UniqueID) 
	as RowNum
from NashvilleHousing 
)
delete from RowNumCTE
where RowNum > 1
-- order by PropertyAddress
```

10. **Dropping Generated Columns:**
    - Removed the temporary columns created during data transformation.

```sql
-- Dropping generated columns

alter table NashvilleHousing 
drop column SoldAsVacentNew, Address, City, Address1,City1,State, OwnerNewAddress, OwnerCity, OwnerState, SoldAsVacantNew

select top 0 * from NashvilleHousing where 1 = 0;
```

## Conclusion:
The NashvilleHousing dataset was imported into SSMS from an Excel file and underwent comprehensive data cleaning and transformation processes. The standardized dataset is now ready for further analysis or reporting tasks.

---

This README format includes the SQL code snippets provided for each step of the data cleaning/transformation project. Adjustments can be made to the SQL code snippets or additional details can be included as needed.
