# 🎵 Chinook Music Store — SQL Analysis Report

![SQLite](https://img.shields.io/badge/Database-SQLite-blue?logo=sqlite)
![SQL](https://img.shields.io/badge/Language-SQL-orange)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![GitHub](https://img.shields.io/badge/Portfolio-Project-purple)

---

## 📌 Introduction

This project is a structured SQL analysis of the **Chinook digital music store database** — a widely used sample database that simulates a real-world music retail business. The analysis was conducted as part of a data analytics portfolio to demonstrate proficiency in database design, SQL querying, and business intelligence reporting.

The Chinook database models a digital media store similar to iTunes, and contains information on **artists, albums, tracks, customers, employees, invoices, and playlists** across multiple countries.

**Tools Used:**
- SQLite (database engine)
- SQL (DDL & DML queries)
- GitHub (version control & documentation)

**Database:** `chinook.db`
**Date Range of Records:** January 2009 – December 2013
**Total Invoices:** 412 | **Total Customers:** 59 | **Countries Covered:** 24

---

## ❓ Problem Statement

The Chinook music store needs to understand its sales performance, customer behaviour, and content popularity to make smarter business decisions. Key business questions include:

- **Which artists and genres generate the most revenue?**
- **Which countries are the highest-spending markets?**
- **Who are the top customers by lifetime spend?**
- **How does monthly revenue trend over time?**
- **Which sales support employees drive the most revenue?**

> **Objective:** Use SQL to extract, analyze, and present actionable insights from the Chinook database that can guide marketing, inventory, and staffing decisions.

---

## 🗄️ Data Sourcing

The data used in this project is the **Chinook Sample Database**, an open-source dataset widely used for SQL learning and portfolio projects.

| Property | Detail |
|----------|--------|
| **Source** | [Chinook Database — GitHub](https://github.com/lerocha/chinook-database) |
| **Format** | SQLite `.db` file |
| **License** | Open Source |
| **Size** | ~1MB |
| **Records** | 412 invoices, 2,240 invoice line items, 3,503 tracks |

### Tables in the Database:

| Table | Description |
|-------|-------------|
| `artists` | Music artists (275 records) |
| `albums` | Albums linked to artists (347 records) |
| `tracks` | Individual tracks with genre, media type, price (3,503 records) |
| `genres` | Music genre categories (25 records) |
| `media_types` | File format types — MP3, AAC, etc. (5 records) |
| `playlists` | Curated playlists (18 records) |
| `playlist_track` | Many-to-many link between playlists and tracks (8,715 records) |
| `customers` | Customer details including country (59 records) |
| `employees` | Staff including sales support agents (8 records) |
| `invoices` | Purchase transactions (412 records) |
| `invoice_items` | Line items per invoice (2,240 records) |

---

## 🔧 Data Transformation and Cleaning

The Chinook database is pre-cleaned and well-structured. However, the following transformations and validations were applied during the analysis phase:

### ✅ Steps Applied:

**1. NULL Value Assessment**
Checked for NULL values in key columns — `customers.Company`, `tracks.Composer`, and `employees.ReportsTo` were found to have NULLs. These were acknowledged as valid sparse fields (not all customers have a company; not all tracks have a known composer), so no imputation was required.

**2. Date Formatting**
`InvoiceDate` was stored as `DATETIME` (e.g., `2009-01-01 00:00:00`). The `strftime()` function was used throughout queries to extract Year, Month, and Quarter for time-series analysis:
```sql
strftime('%Y', InvoiceDate) AS Year,
strftime('%m', InvoiceDate) AS Month
```

**3. Name Concatenation**
Employee and customer full names were constructed by concatenating first and last name fields:
```sql
FirstName || ' ' || LastName AS FullName
```

**4. Revenue Calculation**
Total revenue per invoice line item was derived as a calculated field:
```sql
UnitPrice * Quantity AS LineRevenue
```

**5. JOIN Integrity Validation**
All foreign key relationships were tested to confirm referential integrity. No orphaned records were found across any table joins.

**6. Derived Metrics**
The following metrics were derived using SQL aggregation:
- `TotalRevenue` per artist, genre, and country
- `CustomerLifetimeValue` per customer
- `RevenuePerEmployee` for sales support staff
- `MonthlyRevenueTrend` across the full date range

---

## 🗂️ Data Modeling

The Chinook database follows a well-designed **relational schema** with clearly defined primary and foreign keys across 11 tables.

### Entity Relationship Diagram (ERD):

```
[artists] ──< [albums] ──< [tracks] >── [genres]
                                  |         |
                            [invoice_items]  [media_types]
                                  |
                            [invoices] >── [customers] >── [employees]

[playlists] >──< [playlist_track] >──< [tracks]
```

### Key Relationships:

| Parent Table | Child Table | Foreign Key | Relationship |
|-------------|-------------|-------------|--------------|
| `artists` | `albums` | `ArtistId` | One-to-Many |
| `albums` | `tracks` | `AlbumId` | One-to-Many |
| `genres` | `tracks` | `GenreId` | One-to-Many |
| `media_types` | `tracks` | `MediaTypeId` | One-to-Many |
| `customers` | `invoices` | `CustomerId` | One-to-Many |
| `employees` | `customers` | `SupportRepId` | One-to-Many |
| `invoices` | `invoice_items` | `InvoiceId` | One-to-Many |
| `tracks` | `invoice_items` | `TrackId` | One-to-Many |
| `playlists` | `playlist_track` | `PlaylistId` | Many-to-Many |
| `tracks` | `playlist_track` | `TrackId` | Many-to-Many |

> All relationships follow standard **One-to-Many** or **Many-to-Many** patterns with no circular dependencies.

---

## 📊 Data Analysis and SQL Queries

---

### 📌 Q1 — Top 10 Artists by Revenue

**Business Question:** Which artists generate the most revenue for the store?

```sql
SELECT 
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
```

**Result:**

| Artist | Units Sold | Revenue ($) |
|--------|-----------|-------------|
| Iron Maiden | 140 | 138.60 |
| U2 | 107 | 105.93 |
| Metallica | 91 | 90.09 |
| Led Zeppelin | 87 | 86.13 |
| Lost | 41 | 81.59 |

> 💡 **Insight:** Rock and metal artists dominate revenue. Iron Maiden alone accounts for $138.60 — nearly 30% more than second-place U2.

---

### 📌 Q2 — Top Genres by Revenue

**Business Question:** Which music genres are most popular among buyers?

```sql
SELECT 
    g.Name AS Genre,
    SUM(ii.Quantity) AS TotalSold,
    ROUND(SUM(ii.UnitPrice * ii.Quantity), 2) AS Revenue
FROM invoice_items ii
JOIN tracks t ON ii.TrackId = t.TrackId
JOIN genres g ON t.GenreId = g.GenreId
GROUP BY g.Name
ORDER BY Revenue DESC
LIMIT 10;
```

**Result:**

| Genre | Units Sold | Revenue ($) |
|-------|-----------|-------------|
| Rock | 835 | 826.65 |
| Latin | 386 | 382.14 |
| Metal | 264 | 261.36 |
| Alternative & Punk | 244 | 241.56 |
| TV Shows | 47 | 93.53 |

> 💡 **Insight:** Rock is by far the best-selling genre, generating over twice the revenue of second-place Latin.

---

### 📌 Q3 — Revenue by Country

**Business Question:** Which countries spend the most at the store?

```sql
SELECT 
    BillingCountry,
    COUNT(InvoiceId) AS NumInvoices,
    ROUND(SUM(Total), 2) AS TotalRevenue
FROM invoices
GROUP BY BillingCountry
ORDER BY TotalRevenue DESC
LIMIT 10;
```

**Result:**

| Country | Invoices | Revenue ($) |
|---------|---------|-------------|
| USA | 91 | 523.06 |
| Canada | 56 | 303.96 |
| France | 35 | 195.10 |
| Brazil | 35 | 190.10 |
| Germany | 28 | 156.48 |

> 💡 **Insight:** The USA is the single largest market contributing over $523 — nearly double second-place Canada.

---

### 📌 Q4 — Monthly Revenue Trend

**Business Question:** How does revenue trend month over month?

```sql
SELECT 
    strftime('%Y', InvoiceDate) AS Year,
    strftime('%m', InvoiceDate) AS Month,
    ROUND(SUM(Total), 2) AS MonthlyRevenue
FROM invoices
GROUP BY Year, Month
ORDER BY Year, Month;
```

> 💡 **Insight:** Revenue is relatively stable throughout the dataset period (~$37/month on average), suggesting a consistent but small customer base with room to grow through acquisition campaigns.

---

### 📌 Q5 — Top 10 Customers by Lifetime Spend

**Business Question:** Who are the store's most valuable customers?

```sql
SELECT 
    c.FirstName || ' ' || c.LastName AS Customer,
    c.Country,
    COUNT(i.InvoiceId) AS Purchases,
    ROUND(SUM(i.Total), 2) AS TotalSpent
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
ORDER BY TotalSpent DESC
LIMIT 10;
```

**Result:**

| Customer | Country | Purchases | Total Spent ($) |
|----------|---------|-----------|-----------------|
| Helena Holý | Czech Republic | 7 | 49.62 |
| Richard Cunningham | USA | 7 | 47.62 |
| Luis Rojas | Chile | 7 | 46.62 |
| Ladislav Kovács | Hungary | 7 | 45.62 |
| Hugh O'Reilly | Ireland | 7 | 45.62 |

> 💡 **Insight:** Top customers are spread internationally, showing strong global engagement beyond just the top-revenue countries.

---

### 📌 Q6 — Sales Employee Performance

**Business Question:** Which sales support agent generates the most revenue?

```sql
SELECT 
    e.FirstName || ' ' || e.LastName AS Employee,
    e.Title,
    COUNT(c.CustomerId) AS CustomersSupported,
    ROUND(SUM(i.Total), 2) AS RevenueGenerated
FROM employees e
JOIN customers c ON e.EmployeeId = c.SupportRepId
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY e.EmployeeId
ORDER BY RevenueGenerated DESC;
```

**Result:**

| Employee | Title | Customers | Revenue ($) |
|----------|-------|-----------|-------------|
| Jane Peacock | Sales Support Agent | 146 | 833.04 |
| Margaret Park | Sales Support Agent | 140 | 775.40 |
| Steve Johnson | Sales Support Agent | 126 | 720.16 |

> 💡 **Insight:** Jane Peacock leads in both customer count and revenue. The performance gap could inform training or incentive programs.

---

## ✅ Conclusions and Recommendations

Based on the SQL analysis of the Chinook music store database:

**1. Double down on Rock & Metal marketing**
Rock generates over 3× more revenue than any other genre. Promotional campaigns, featured playlists, and curated recommendations should prioritize this genre.

**2. Expand the US customer base**
The USA leads in revenue but represents a single market. Targeted loyalty programs could significantly boost US retention and referrals.

**3. Reward top customers globally**
Top spenders come from Czech Republic, Chile, Hungary, and Ireland — not just the USA. A global VIP loyalty program could improve retention across all markets.

**4. Investigate underperforming genres**
Genres like Jazz and Classical have relatively low revenue despite cultural prestige. Curation improvements or pricing adjustments could close this gap.

**5. Study top sales agent performance**
Jane Peacock outperforms peers by $57–$113. Her approach could be documented and shared to lift overall team performance.

---

## 📁 Repository Structure

```
📦 Chinook-SQL-Analysis
 ┣ 📂 database
 ┃ ┗ 📄 chinook.db
 ┣ 📂 queries
 ┃ ┣ 📄 01_top_artists.sql
 ┃ ┣ 📄 02_top_genres.sql
 ┃ ┣ 📄 03_revenue_by_country.sql
 ┃ ┣ 📄 04_monthly_revenue.sql
 ┃ ┣ 📄 05_top_customers.sql
 ┃ ┗ 📄 06_employee_performance.sql
 ┣ 📄 README.md
```

---

## 👤 Author

**Your Name**
📧 youremail@example.com
🔗 www.linkedin.com/in/osebhaheme-etumah-357678199 | [[GitHub](https://github.com)
](https://github.com/Osebhaheme-18)
---

