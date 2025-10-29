# Advanced-Data-Management
Code created for D326 @ WGU

Task 1 is about determining a business question that’ll draw data from multiple tables. The Question that I wish to ask is simply, how many movies in total do we rent every month? This question is straight forward and can help benefit the business by giving them and exact number of sales. It is also the first step into analyzing how the stores are successful.



A.  Summarize one real-world written business report that can be created from the DVD Dataset from the “Labs on Demand Assessment Environment and DVD Database” attachment. 
1.  Identify the specific fields that will be included in the detailed table and the summary table of the report.
The Detailed table will include the following fields: rental_total(INT), staff_id(INT), rental_location(VARCHAR), rental_month(INT), rental_year(INT)

The Summary table will include the following fields: rental_total(INT), rental_month(INT), rental_year(INT)

2.  Describe the types of data fields used for the report.
3.  
Integer for the whole numbers associated with the rental total, employee IDs, and the date. With VARCHAR for where the movies were rented from. I will also have it separated by the month and year which will be expressed as integers.

4.  Identify at least two specific tables from the given dataset that will provide the data necessary for the detailed table section and the summary table section of the report.

For the detailed and summary tables I will be pulling data from the Rental and Address tables.

4.  Identify at least one field in the detailed table section that will require a custom transformation with a user-defined function and explain why it should be transformed (e.g., you might translate a field with a value of N to No and Y to Yes).
5.  
I will create two transformations for changing the date which is a timestamp into a month and year. Which I can then display in my created tables.

6.  Explain the different business uses of the detailed table section and the summary table section of the report.
7.  
The summary table can be used for an easy side by side comparison for the company as a whole to see the total rental amount.
The detailed table would be good to understand where more rentals are being purchased. If a certain staff member or rental location is doing better than others, they can then be studied to understand why they might be more effective.

8.  Explain how frequently your report should be refreshed to remain relevant to stakeholders.
9.  
It should be regularly refreshed each month. You can use it every week which was my first thought but to get a complete understanding on how the stores perform it’d need to be run every month.. Due to location and a number of variable factors a store could do better one week than another.  Thus it is better to wait the whole month to compare data.
 
B.  Provide original code for function(s) in text format that perform the transformation(s) you identified in part A4.

CREATE OR REPLACE FUNCTION rental_month(rental_date timestamp)
RETURNS int
LANGUAGE plpgsql
AS 
$$
DECLARE month_of_sale int;
BEGIN
	SELECT EXTRACT(MONTH FROM rental_date) INTO month_of sale;
	RETURN month_of_sale;
END;
$$

CREATE OR REPLACE FUNCTION rental_year(rental_date timestamp)
RETURNS int
LANGUAGE plpgsql
AS 
$$
DECLARE year_of_sale int;
BEGIN
	SELECT EXTRACT(YEAR FROM rental_date) INTO year_of sale;
	RETURN year_of_sale;
END;
$$



 
C.  Provide original SQL code in a text format that creates the detailed and summary tables to hold your report table sections.

CREATE TABLE summary_table (
rental_total INT,
rental_month  INT,
rental_year  INT
)

CREATE TABLE detailed_table (
rental_total INT,
staff_id INT,
rentalLocation INT,
rental_month  INT,
rental_year  INT
)
 
D.  Provide an original SQL query in a text format that will extract the raw data needed for the detailed section of your report from the source database.


	INSERT INTO summary_table
SELECT 
COUNT(rental_id) AS rental_total,
	rental_month(rental_date),
	rental_year(rental_date)
FROM rental
GROUP BY
	rental_year(rental_date),
	rental_month(rental_date)
ORDER BY 
	rental_year(rental_date),
	rental_month(rental_date);

INSERT INTO detailed_table
SELECT 
    COUNT(r.rental_id) AS rental_total,
    r.staff_id,
    a.address AS rentalLocation,
    rental_month(r.rental_date),
    rental_year(r.rental_date)
FROM rental r
JOIN staff s ON r.staff_id = s.staff_id
JOIN address a ON s.address_id = a.address_id
GROUP BY
    r.staff_id,
    a.address,
    rental_year(r.rental_date),
    rental_month(r.rental_date)
ORDER BY 
    rental_year(r.rental_date),
    rental_month(r.rental_date);


 
E.  Provide original SQL code in a text format that creates a trigger on the detailed table of the report that will continually update the summary table as data is added to the detailed table.

CREATE OR REPLACE FUNCTION update_summary_from_detailed()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Remove old row for this month/year (if there is one)
    DELETE FROM summary_table
    WHERE rental_month = NEW.rental_month
      AND rental_year = NEW.rental_year;

    -- Insert updated row
    INSERT INTO summary_table (rental_total, rental_month, rental_year)
    SELECT 
        SUM(rental_total),
        NEW.rental_month,
        NEW.rental_year
    FROM detailed_table
    WHERE rental_month = NEW.rental_month
      AND rental_year = NEW.rental_year;

    RETURN NEW;
END;
$$

--create trigger on detailed_table
CREATE TRIGGER trg_update_summary
AFTER INSERT ON detailed_table
FOR EACH ROW
EXECUTE FUNCTION update_summary_from_detailed();






F.  Provide an original stored procedure in a text format that can be used to refresh the data in both the detailed table and summary table. The procedure should clear the contents of the detailed table and summary table and perform the raw data extraction from part D.

CREATE OR REPLACE PROCEDURE refresh_rental_reports()
LANGUAGE plpgsql
AS $$
BEGIN
    -- clear old data
    TRUNCATE TABLE detailed_table;
    TRUNCATE TABLE summary_table;

    -- reinsert data into detailed_table
    INSERT INTO detailed_table (rental_total, staff_id, rentalLocation, rental_month, rental_year)
    SELECT 
        COUNT(r.rental_id) AS rental_total,
        r.staff_id,
        a.address AS rentalLocation,
        rental_month(r.rental_date),
        rental_year(r.rental_date)
    FROM rental r
    JOIN staff s ON r.staff_id = s.staff_id
    JOIN address a ON s.address_id = a.address_id
    GROUP BY
        r.staff_id,
        a.address,
        rental_year(r.rental_date),
        rental_month(r.rental_date)
    ORDER BY 
        rental_year(r.rental_date),
        rental_month(r.rental_date)

END;
$$
1.  Identify a relevant job scheduling tool that can be used to automate the stored procedure.
I would use pgAgent, it is able to run scripts and create ‘jobs’ that automate tasks. I can use it to then run the procedure whenever the company would like.
