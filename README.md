# Data Cleaning in SQL Queries

## Introduction
- This set of SQL queries is designed to clean and standardize data in the NashvilleHousing dataset.

## Table of Contents
- [Standardize Date Format](#standardize-date-format)
- [Populate Property Address Data](#populate-property-address-data)
- [Breaking out Address into Individual Columns](#breaking-out-address-into-individual-columns)
- [Breaking out Owner Address into Individual Columns](#breaking-out-owner-address-into-individual-columns)
- [Change Y and N to Yes and No](#change-y-and-n-to-yes-and-no)
- [Remove Duplicates](#remove-duplicates)
- [Delete Unused Columns](#delete-unused-columns)

## Standardize Date Format
- This section standardizes the SaleDate column to the Date format.

        Select SaleDateConverted, CONVERT(Date,SaleDate)
        From PortfolioProject.dbo.NashvilleHousing
        
        
        Update NashvilleHousing
        SET SaleDate = CONVERT(Date,SaleDate)
        
        -- If it doesn't Update properly
        
        ALTER TABLE NashvilleHousing
        Add SaleDateConverted Date;
    
        -- Set the new column to the converted SaleDate
        
        Update NashvilleHousing
        SET SaleDateConverted = CONVERT(Date,SaleDate)


## Populate Property Address Data
- This section populates missing PropertyAddress values based on non-null values from other records with the same ParcelID.

        -- Populate Property Address data
        Select *
        From PortfolioProject.dbo.NashvilleHousing
        --Where PropertyAddress is null
        order by ParcelID
        
        
        -- Update PropertyAddress with non-null values
        Select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
        From PortfolioProject.dbo.NashvilleHousing a
        JOIN PortfolioProject.dbo.NashvilleHousing b
        	on a.ParcelID = b.ParcelID
        	AND a.[UniqueID ] <> b.[UniqueID ]
        Where a.PropertyAddress is null
        
        
        Update a
        SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
        From PortfolioProject.dbo.NashvilleHousing a
        JOIN PortfolioProject.dbo.NashvilleHousing b
        	on a.ParcelID = b.ParcelID
        	AND a.[UniqueID ] <> b.[UniqueID ]
        Where a.PropertyAddress is null

## Breaking out Address into Individual Columns
- This section breaks the PropertyAddress column into individual Address and City columns.

        -- Breaking out Owner Address into Individual Columns
        Select PropertyAddress
        From PortfolioProject.dbo.NashvilleHousing
        --Where PropertyAddress is null
        --order by ParcelID
        
        SELECT
        SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 ) as Address
        , SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1 , LEN(PropertyAddress)) as Address
        
        From PortfolioProject.dbo.NashvilleHousing
        
        
        ALTER TABLE NashvilleHousing
        Add PropertySplitAddress Nvarchar(255);
        
        Update NashvilleHousing
        SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 )
        
        
        ALTER TABLE NashvilleHousing
        Add PropertySplitCity Nvarchar(255);
        
        Update NashvilleHousing
        SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1 , LEN(PropertyAddress))
        
        
        
        
        Select *
        From PortfolioProject.dbo.NashvilleHousing
        
        
        
        
        
        Select OwnerAddress
        From PortfolioProject.dbo.NashvilleHousing
        
        
        Select
        PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)
        ,PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2)
        ,PARSENAME(REPLACE(OwnerAddress, ',', '.') , 1)
        From PortfolioProject.dbo.NashvilleHousing
        
        
        
        ALTER TABLE NashvilleHousing
        Add OwnerSplitAddress Nvarchar(255);
        
        Update NashvilleHousing
        SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)
        
        
        ALTER TABLE NashvilleHousing
        Add OwnerSplitCity Nvarchar(255);
        
        Update NashvilleHousing
        SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2)
        
        
        
        ALTER TABLE NashvilleHousing
        Add OwnerSplitState Nvarchar(255);
        
        Update NashvilleHousing
        SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 1)
        
        
        
        Select *
        From PortfolioProject.dbo.NashvilleHousing

## Change Y and N to Yes and No
- This section updates the "Sold as Vacant" field to use "Yes" and "No" instead of "Y" and "N".

        -- Change Y and N to Yes and No in "Sold as Vacant" field
        Select Distinct(SoldAsVacant), Count(SoldAsVacant)
        From PortfolioProject.dbo.NashvilleHousing
        Group by SoldAsVacant
        
        Select SoldAsVacant
        , CASE When SoldAsVacant = 'Y' Then 'Yes'
        	When SoldAsVacant = 'N' Then 'No'
        	Else SoldAsVacant
        	END
        From PortfolioProject..NashvilleHousing
        
        Update NashvilleHousing
        SET SoldAsVacant = CASE When SoldAsVacant = 'Y' Then 'Yes'
        	When SoldAsVacant = 'N' Then 'No'
        	Else SoldAsVacant
        	END

## Remove Duplicates
- This section removes duplicate records based on specific columns.

        -- Remove Duplicates
        WITH RowNumCTE as(
        Select *,
        	ROW_NUMBER() OVER (
        	PARTITION BY ParcelID,
        				PropertyAddress,
        				SalePrice,
        				SaleDate,
        				LegalReference
        				ORDER BY 
        					UniqueID
        					) as row_num
        From PortfolioProject..NashvilleHousing
        --Order by ParcelID
        )
        Select * -- Delete statement replaced w/ Select * to check
        From RowNumCTE
        Where row_num > 1
        --Order by PropertyAddress

## Delete Unused Columns
- This section removes unused columns from the dataset.

        -- Delete Unused Columns
        Select *
        From PortfolioProject..NashvilleHousing
        
        
        ALTER TABLE PortfolioProject..NashvilleHousing
        DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress
        
        ALTER TABLE PortfolioProject..NashvilleHousing
        DROP COLUMN Saledate
