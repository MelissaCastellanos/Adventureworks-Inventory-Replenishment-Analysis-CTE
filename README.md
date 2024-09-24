# CTE Applications
## What is a CTE?  
![CTE Syntax](https://learnsql.com/blog/sql-common-table-expression-guide/cte_syntax_example.jpg)  
  
CTE's are temporary result sets that are defined before a SELECT statement and execute with the SELECT statement. These tables can be refrenced within the same query and are extremely useful tools when querying complex data.
They are easier read and work with when compared to subquieres, and are an essental tool.

## Inventory Replenishment Analysis CTE  
Below is a simple CTE used to extract product details to determine which products need to be reordered. The CTE JOINS the tables that contain the necessary product details, while the SELECT statment then filters down products whose stock is equal to or below the reorder point to determine what needs to be reordered.  
```
--SQL
with 
LowInventoryProducts as 
(
SELECT 
	P.[EnglishProductName],
	P.[SafetyStockLevel],
	P.[ReorderPoint],
	P.[DaysToManufacture],
	PI.[MovementDate],
	PI.[UnitsBalance]
FROM [AdventureWorksDW2022].[dbo].[FactProductInventory] AS PI
JOIN [AdventureWorksDW2022].[dbo].[DimProduct] AS P 
ON PI.[ProductKey] = P.[ProductKey] 
)
SELECT *
FROM LowInventoryProducts
WHERE UnitsBalance <= ReorderPoint

```

## Top 5 Selling Products by Category CTE

```
--SQL
with  
InternetSales AS
(
	SELECT 
		I.[ProductKey],
		I.[SalesAmount],
		P.[EnglishProductName],
		P.[Color],
		P.[Size],
		P.[SizeRange],
		P.[Weight],
		P.[ProductLine],
		P.[Class],
		P.[Style],
		P.[ModelName],
		PS.[EnglishProductSubcategoryName],
		PS.[ProductCategoryKey]
	FROM [AdventureWorksDW2022].[dbo].[FactInternetSales] AS I
	LEFT JOIN [AdventureWorksDW2022].[dbo].[DimProduct] AS P
	ON I.ProductKey = P.ProductKey
	LEFT JOIN [AdventureWorksDW2022].[dbo].[DimProductSubcategory] as PS
	ON P.ProductSubcategoryKey = PS.ProductSubcategoryKey
),
Ranking AS
(
SELECT 
	ROUND(SUM([SalesAmount]),2) AS TotalSales,
	RANK () OVER (PARTITION BY [ProductCategoryKey] ORDER BY SUM([SalesAmount]) DESC) AS SalesRanked,
	[ProductCategoryKey],
	[EnglishProductSubcategoryName],
	[ProductKey]
FROM InternetSales
GROUP BY [ProductCategoryKey], [EnglishProductSubcategoryName], [Color], [ProductKey]
)
SELECT DISTINCT
	R.[SalesRanked],
	R.[ProductCategoryKey],
	R.[EnglishProductSubcategoryName],
	I.[SalesAmount],
	R.[TotalSales],
	R.[ProductKey],
	I.[EnglishProductName],
	I.[Color],
	I.[Size],
	I.[SizeRange],
	I.[Weight],
	I.[ProductLine],
	I.[Class],
	I.[Style],
	I.[ModelName]
FROM InternetSales AS I
INNER JOIN Ranking AS R
ON I.[ProductKey] = R.[ProductKey]
WHERE SalesRanked <=5
ORDER BY ProductCategoryKey,SalesRanked
```
