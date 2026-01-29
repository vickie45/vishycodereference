# SQL Server Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable T-SQL snippets, best practices, and real-world use cases for Microsoft SQL Server.

---

## Table of contents
- Setup & tools
- Basic T-SQL syntax
- Data types
- DDL (tables, schemas, indexes)
- DML (SELECT, INSERT, UPDATE, DELETE)
- Joins & set operations
- Aggregation & window functions
- Transactions & isolation levels
- Indexing & statistics
- Query tuning & execution plans
- Stored procedures, functions & views
- Triggers
- Security & permissions
- Backups, restore & HA/DR
- Monitoring & DMVs
- Maintenance & best practices
- Practical examples / scenarios

---

## Setup & tools
- Install: SQL Server (Express/Developer) or container image: mcr.microsoft.com/mssql/server
- Tools: SQL Server Management Studio (SSMS), Azure Data Studio, sqlcmd, bcp
- Connect string example: Server=.;Database=MyDb;User Id=sa;Password=YourPassword;

---

## Basic T-SQL syntax
- Statement terminator: optional semicolon (recommended)
- Basic SELECT:
```sql
SELECT TOP 10 Id, Name FROM dbo.Products WHERE IsActive = 1 ORDER BY CreatedAt DESC;
```
- Variables & control:
```sql
DECLARE @count INT = 0;
IF @count = 0 BEGIN SET @count = 1; END
```

---

## Data types (common)
- Numeric: INT, BIGINT, DECIMAL(p,s), FLOAT
- Date/time: DATE, TIME, DATETIME2, DATETIMEOFFSET
- Strings: VARCHAR(n), NVARCHAR(n), CHAR, NCHAR, TEXT (deprecated)
- Binary: VARBINARY
- Special: BIT, UNIQUEIDENTIFIER, XML, JSON (stored as NVARCHAR)
Best practice: choose appropriate sizes; use NVARCHAR for Unicode only when needed.

---

## DDL — tables, constraints, schemas
- Create table:
```sql
CREATE SCHEMA sales;
CREATE TABLE sales.Orders (
  OrderId INT IDENTITY(1,1) PRIMARY KEY,
  CustomerId INT NOT NULL,
  Total DECIMAL(18,2) NOT NULL,
  CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);
```
- Add constraint/index:
```sql
ALTER TABLE sales.Orders ADD CONSTRAINT FK_Orders_Cust FOREIGN KEY (CustomerId) REFERENCES dbo.Customers(Id);
CREATE INDEX IX_Orders_CustomerId ON sales.Orders(CustomerId);
```

---

## DML — insert, update, delete, merge
- Insert:
```sql
INSERT INTO dbo.Products(Name, Price) VALUES ('Pen', 1.25);
INSERT INTO dbo.Products(Name, Price) OUTPUT inserted.Id VALUES ('Book', 9.99);
```
- Update/Delete:
```sql
UPDATE dbo.Products SET Price = Price * 1.1 WHERE CategoryId = 2;
DELETE FROM dbo.Sessions WHERE ExpiresAt < SYSUTCDATETIME();
```
- MERGE (careful with concurrency):
```sql
MERGE INTO dbo.Target AS T
USING (VALUES(1,'X')) AS S(Id, Val)
ON T.Id = S.Id
WHEN MATCHED THEN UPDATE SET Val = S.Val
WHEN NOT MATCHED THEN INSERT (Id, Val) VALUES (S.Id, S.Val);
```

---

## Joins & set operations
- INNER, LEFT, RIGHT, FULL, CROSS JOIN
```sql
SELECT o.OrderId, c.Name
FROM sales.Orders o
JOIN dbo.Customers c ON o.CustomerId = c.Id
LEFT JOIN dbo.Payments p ON p.OrderId = o.OrderId;
```
- UNION / UNION ALL, INTERSECT, EXCEPT

---

## Aggregation & window functions
- Aggregates:
```sql
SELECT CustomerId, COUNT(*) AS Orders, SUM(Total) AS TotalSpent
FROM sales.Orders GROUP BY CustomerId HAVING SUM(Total) > 100;
```
- Window functions:
```sql
SELECT OrderId, Total,
  SUM(Total) OVER (PARTITION BY CustomerId ORDER BY CreatedAt ROWS UNBOUNDED PRECEDING) AS RunningTotal,
  ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY CreatedAt DESC) AS RowNum
FROM sales.Orders;
```

---

## Transactions & isolation levels
- Control:
```sql
BEGIN TRAN;
-- statements
COMMIT TRAN;
ROLLBACK TRAN;
```
- Isolation levels: READ UNCOMMITTED, READ COMMITTED (default), REPEATABLE READ, SNAPSHOT, SERIALIZABLE
- Use SNAPSHOT or appropriate level to avoid blocking; beware of locking behavior.

---

## Indexing & statistics
- Clustered vs nonclustered: clustered defines physical order; only one per table.
- Covering index: include columns to avoid lookups.
```sql
CREATE NONCLUSTERED INDEX IX_Orders_Cust_Total ON sales.Orders(CustomerId) INCLUDE (Total, CreatedAt);
```
- Update statistics:
```sql
UPDATE STATISTICS dbo.Orders WITH FULLSCAN;
```
Best practice: index selective columns, monitor fragmentation, avoid over-indexing for heavy write tables.

---

## Query tuning & execution plans
- Use Actual Execution Plan (SSMS) or SET STATISTICS IO/TIME ON.
```sql
SET STATISTICS IO ON; SET STATISTICS TIME ON;
-- run query
SET STATISTICS IO OFF; SET STATISTICS TIME OFF;
```
- Look for:
  - High logical/physical reads
  - Table/index scans vs seeks
  - Expensive operators (Hash Match, Sort)
- Use Query Store (built-in) to track regressions.

---

## Stored procedures, functions & views
- Stored proc:
```sql
CREATE PROCEDURE dbo.CreateOrder @CustomerId INT, @Total DECIMAL(18,2)
AS
BEGIN
  SET NOCOUNT ON;
  INSERT INTO sales.Orders(CustomerId, Total) VALUES(@CustomerId, @Total);
  SELECT SCOPE_IDENTITY() AS NewId;
END
```
- Scalar/table-valued functions (inline TVFs preferred for performance).
- Views: use for logical abstraction; indexed views for heavy aggregations (with restrictions).

---

## Triggers
- AFTER / INSTEAD OF triggers for audit or complex business rules.
```sql
CREATE TRIGGER trg_AuditOrders ON sales.Orders AFTER INSERT, UPDATE
AS
  INSERT INTO dbo.OrdersAudit(OrderId, ActionAt) SELECT Id, SYSUTCDATETIME() FROM inserted;
```
Best practice: keep triggers light; avoid heavy processing.

---

## Security & permissions
- Principle of least privilege; use roles, not sa. Use contained users or Azure AD when possible.
- Grant/revoke:
```sql
CREATE USER appUser WITH PASSWORD = 'StrongP@ss';
ALTER ROLE db_datareader ADD MEMBER appUser;
GRANT EXECUTE ON SCHEMA::dbo TO appUser;
```
- Always parameterize queries to avoid SQL injection; prefer stored procs or parameterized commands.

---

## Backups, restore & HA/DR
- Backup:
```sql
BACKUP DATABASE MyDb TO DISK = 'C:\backups\MyDb.bak' WITH COMPRESSION;
```
- Restore:
```sql
RESTORE DATABASE MyDb FROM DISK = 'C:\backups\MyDb.bak' WITH RECOVERY, MOVE 'MyDb_Data' TO 'D:\Data\MyDb.mdf';
```
- HA options: Always On Availability Groups, Failover Clustering, Log Shipping, Replication (use-case dependent).
- Test restores regularly.

---

## Monitoring & DMVs
- Useful DMVs:
  - sys.dm_exec_requests, sys.dm_exec_query_stats, sys.dm_exec_sql_text(sql_handle)
  - sys.dm_db_index_physical_stats, sys.dm_db_index_usage_stats
  - sys.dm_os_wait_stats
- Example find top expensive queries:
```sql
SELECT TOP 10
  qs.total_logical_reads, qs.total_worker_time, SUBSTRING(st.text, (qs.statement_start_offset/2)+1, ((qs.statement_end_offset-qs.statement_start_offset)/2)+1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY qs.total_logical_reads DESC;
```

---

## Maintenance & best practices
- Regular maintenance plan: index rebuild/reorganize, update statistics, integrity check (DBCC CHECKDB)
- Avoid auto-shrink, schedule heavy maintenance during low traffic
- Use appropriate recovery model (FULL for point-in-time, SIMPLE for less logging)
- Use parameterization and plan guides only when needed

---

## Practical examples & scenarios

1) Pagination (efficient)
```sql
-- keyset pagination using CreatedAt
SELECT TOP(20) * FROM sales.Orders WHERE (CustomerId = @cid) AND (CreatedAt < @lastCreatedAt) ORDER BY CreatedAt DESC;
```

2) Upsert with OUTPUT
```sql
MERGE INTO dbo.Products AS T
USING (VALUES (@Sku, @Name, @Price)) AS S(Sku, Name, Price)
ON T.Sku = S.Sku
WHEN MATCHED THEN UPDATE SET Name = S.Name, Price = S.Price
WHEN NOT MATCHED THEN INSERT (Sku, Name, Price) VALUES (S.Sku, S.Name, S.Price)
OUTPUT $action, inserted.Id;
```

3) Parameterized query example (client)
- Always use parameters, not string concatenation:
```sql
-- C# ADO.NET example (concept)
using var cmd = new SqlCommand("SELECT * FROM dbo.Products WHERE CategoryId = @cat", conn);
cmd.Parameters.AddWithValue("@cat", categoryId);
```

4) Safe schema changes
- Add nullable column, backfill in batches, then make non-nullable:
```sql
ALTER TABLE dbo.Users ADD IsActive BIT NULL;
UPDATE dbo.Users SET IsActive = 1 WHERE ...; -- batched
ALTER TABLE dbo.Users ALTER COLUMN IsActive BIT NOT NULL;
```

---

## Quick reference one-liners
- Current UTC time: SYSUTCDATETIME()
- Row count estimate: sys.dm_db_partition_stats
- Disable index: ALTER INDEX ALL ON dbo.TableName DISABLE;
- Rebuild index: ALTER INDEX IX_Name ON dbo.TableName REBUILD;

---

## Further reading / next steps
- Deep dive: Execution plan operators, optimizer statistics, parameter sniffing remedies, Query Store, Query Tuning Advisor
- Learn features: In-Memory OLTP, Columnstore indexes, Temporal tables, Stretch Database (deprecated in some contexts)
- Regularly consult Microsoft Docs and keep SQL Server patched.

---

Keep this file as your SQL Server single-page quick reference. Expand real-world snippets as you encounter patterns in projects.
