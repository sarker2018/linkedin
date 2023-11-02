# Recursion inn SQL
  (2023-11-02)

```sql
/*
	Create a sample DB, a sample table, and populate the table.
	For my case I used an SQL server instance on my local machine.
	You can also use Azure SQL if you do not want to install locally.
*/

CREATE DATABASE Hotels;

USE Hotels;

DROP TABLE IF EXISTS Bookings;

CREATE TABLE Bookings (
    ID INT PRIMARY KEY,
	date_in DATE,
	date_out DATE,
    hotel varchar(255)
);

TRUNCATE TABLE Bookings; -- use when want repopulate with new entries

INSERT INTO Bookings
	(ID, date_in, date_out, hotel)
	VALUES
	(1, '2023-01-10', '2023-01-12', 'room_a'), -- 3 days
	(2, '2023-02-10', '2023-02-12', 'room_b'); -- 3 days
	-- '2023-06-01' fur 120 days
SELECT *
FROM Bookings;


/*
	Now expand the dates by using RECURSION
	As per entries in the table, we should expect total 6 rows:
	3 for each unique IDs.
*/

WITH booking AS 
(
SELECT 
	ID
	, date_in as date
	, date_in
	, date_out
	, hotel
	FROM Bookings
	UNION ALL
	SELECT 
	bns.ID
	, DATEADD(DAY, 1, bns.date)
	, bns.date_in
	, bns.date_out
	, bns.hotel
	FROM booking AS bns
	WHERE DATEADD(DAY, 1, bns.date) <= bns.date_out
)
SELECT *
FROM booking
ORDER BY ID

/*
	For this scenario I used short term booking: i.e. 3 days bookings.
	What would be the scenario, if an entry has a long-time booking, i.e. more than 100 days?
*/

TRUNCATE TABLE Bookings; -- use when you need delete & repopulate with new entries

INSERT INTO Bookings
	(ID, date_in, date_out, hotel)
	VALUES
	(1, '2023-01-10', '2023-01-12', 'room_a'), -- 3 days
	(2, '2023-02-10', '2023-06-10', 'room_b'); -- 121 days

SELECT *
FROM Bookings;
/*
	run the recusrion again
*/

WITH booking AS 
(
SELECT 
	ID
	, date_in as date
	, date_in
	, date_out
	, hotel
	FROM Bookings
	UNION ALL
	SELECT 
	bns.ID
	, DATEADD(DAY, 1, bns.date)
	, bns.date_in
	, bns.date_out
	, bns.hotel
	FROM booking AS bns
	WHERE DATEADD(DAY, 1, bns.date) <= bns.date_out
)
SELECT *
FROM booking
ORDER BY ID

/*
	Opps! Got an error. SQL Server cannot handle number of recursion more than 100 (default)
	There is a solution!
*/

WITH booking AS 
(
SELECT 
	ID
	, date_in as date
	, date_in
	, date_out
	, hotel
	FROM Bookings
	UNION ALL
	SELECT 
	bns.ID
	, DATEADD(DAY, 1, bns.date)
	, bns.date_in
	, bns.date_out
	, bns.hotel
	FROM booking AS bns
	WHERE DATEADD(DAY, 1, bns.date) <= bns.date_out
)
SELECT *
FROM booking
ORDER BY ID
OPTION (MAXRECURSION 0) -- this line will make the number of recursion dynamic!

/*
	As expected, we should get total 124 rows:
	3 rows for the first short-term booking &
	121 rows for second long-term booking  											
*/



```
