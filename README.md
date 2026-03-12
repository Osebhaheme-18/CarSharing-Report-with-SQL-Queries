# CarSharing-Report-with-SQL-Queries
## Introduction
This project is a structured SQL analysis of the Chinook digital music store database a widely used sample database that simulates a real-world music retail business. The analysis was conducted as part of a data analytics portfolio to demonstrate proficiency in database design, SQL querying, and business intelligence reporting.
The Chinook database models a digital media store similar to iTunes, and contains information on artists, albums, tracks, customers, employees, invoices, and playlists across multiple countries.
Tools Used:
- SQLite (database engine)
- Googlesheets
- GitHub ( documentation)
Database: chinook.db
Date Range of Records: January 2009 – December 2013
Total Invoices: 412 | Total Customers: 59 | Countries Covered: 24
## ❓Problem Statement
The Chinook music store needs to understand its sales performance, customer behaviour, and content popularity to make smarter business decisions. Key business questions include:
- Which artists and genres generate the most revenue?
- Which countries are the highest-spending markets?
- Who are the top customers by lifetime spend?
- How does monthly revenue trend over time?
- Which sales support employees drive the most revenue?
## 🗄️ Data Sourcing
The data used in this project is the Chinook Sample Database, an open-source dataset widely used for SQL learning and portfolio projects.
PropertyDetailSourceChinook Database — GitHubFormatSQLite .db fileLicenseOpen SourceSize~1MBRecords412 invoices, 2,240 invoice line items, 3,503 tracks
Tables in the Database:
TableDescriptionartistsMusic artists (275 records)albumsAlbums linked to artists (347 records)tracksIndividual tracks with genre, media type, price (3,503 records)genresMusic genre categories (25 records)media_typesFile format types — MP3, AAC, etc. (5 records)playlistsCurated playlists (18 records)playlist_trackMany-to-many link between playlists and tracks (8,715 records)customersCustomer details including country (59 records)employeesStaff including sales support agents (8 records)invoicesPurchase transactions (412 records)invoice_itemsLine items per invoice (2,240 records)

## 🔧 Data Transformation and Cleaning
The Chinook database is pre-cleaned and well-structured. However, the following transformations and validations were applied during the analysis phase:
✅ Steps Applied:
1. NULL Value Assessment
Checked for NULL values in key columns — customers.Company, tracks.Composer, and employees.ReportsTo were found to have NULLs. These were acknowledged as valid sparse fields (not all customers have a company; not all tracks have a known composer), so no imputation was required.
2. Date Formatting
InvoiceDate was stored as DATETIME (e.g., 2009-01-01 00:00:00). The strftime() function was used throughout queries to extract Year, Month, and Quarter for time-series analysis:
sqlstrftime('%Y', InvoiceDate) AS Year,
strftime('%m', InvoiceDate) AS Month
3. Name Concatenation
Employee and customer full names were constructed by concatenating first and last name fields:
sqlFirstName || ' ' || LastName AS FullName
4. Revenue Calculation
Total revenue per invoice line item was derived as a calculated field:
sqlUnitPrice * Quantity AS LineRevenue
5. JOIN Integrity Validation
All foreign key relationships were tested to confirm referential integrity. No orphaned records were found across any table joins.
6. Derived Metrics
The following metrics were derived using SQL aggregation:

TotalRevenue per artist, genre, and country
CustomerLifetimeValue per customer
RevenuePerEmployee for sales support staff
MonthlyRevenueTrend across the full date range
## 🗂️ Data Modeling
The Chinook database follows a well-designed relational schema with clearly defined primary and foreign keys across 11 tables.
Entity Relationship Diagram (ERD):
[artists] ──< [albums] ──< [tracks] >── [genres]

                                  |         |
                                  
                            [invoice_items]  [media_types]
                            
                                  |
                                  
                            [invoices] >── [customers] >── [employees]
                            
[playlists] >──< [playlist_track] >──< [tracks]


Key Relationships:
Parent Table    Child Table    Foreign Key    Relationship
artists         albums          ArtistId      One-to-Many

All relationships follow standard One-to-Many or Many-to-Many patterns with no circular dependencies.
## 📊 Data Analysis and SQL Queries

📌 Q1 — Top 10 Artists by Revenue
Business Question: Which artists generate the most revenue for the store?
sqlSELECT 
    ar.Name AS Artist,
    SUM(ii.Quantity) AS TotalUnitsSold,
    ROUND(SUM(ii.UnitPrice * ii.Quantity), 2) AS Revenue
FROM invoice_items ii
JOIN tracks t ON ii.TrackId = t.TrackId
JOIN albums al ON t.AlbumId = al.AlbumId
JOIN artists ar ON al.ArtistId = ar.ArtistId
GROUP BY ar.Name
ORDER BY Revenue DESC
LIMIT 10;











