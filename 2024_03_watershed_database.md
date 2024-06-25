## DATA EXTRACTION FROM DATABASE FOR ANALYSIS 

For this project, I used MySQL to extract data from a database that included some properties' records from Watershed Properties used as long\-term rentals, regarding their location and building characteristics, as well as yearly profits and occupancy rate. Another table included records from some other properties used as short term rentals, including their characteristics and rates per night.

Through SQL queries, the data was explored and transformed before being merged into a single table needed for the analysis completion and calculate the estimated increase in profits needed to decide which properties should be converted into short-term rentals. This analysis was completed on Excel and Tableau. \(you can [read more here](https://www.datascienceportfol.io/lugmenn/projects/0)\).

![watershed schema](assets/p06- data cleansing excel/WSP_schema.png "Database Schema")

These are some of the queries ran on MySQL to complete the data extraction and part of its transformation.

### EXPLORING THE AVAILABLE DATA

Reviewed each of the tables' fields contained in them.

```sql
SHOW columns FROM capstone.location;

SHOW columns FROM capstone.property_type;

SHOW columns FROM capstone.st_property_info;

SHOW columns FROM capstone.st_rental_dates;

SHOW columns FROM capstone.st_rental_prices;

SHOW columns FROM capstone.watershed_property_info;
```


I explored the structure of the data contained in each of the tables in order to know if data needed to be cleaned and transformed. Additionally, since it was of interest to get the location of each porperty, I reviewed if there were any NULL Values in the _state_ field and summarized how many records where on each state.


```sql
-- query used for the location table. Similar queries were used in the rest of the tables in the database
SELECT * 
FROM capstone.location
LIMIT 50;


SELECT DISTINCT 
		state,
		COUNT(location_id) as Number_of_records
FROM capstone.location
GROUP BY state;

```

### DATA TRANSFORMATION AND JOINING TABLES

After getting to comprehend better the available data, I decided the available short-term rental properties data (_st_property_) should be unified.
First, it was necessary to fully understand the records on the _rental_date_ field.

```sql
SELECT
    rental_date,
    st_property	
FROM capstone.st_rental_dates
WHERE st_property LIKE '%ST100%';

```

The output showed some records contained consecutive days, while some other didn't. This meant _rental_date_ referred to the specific day(s) a property was rented for a short-term stay, either a single night or more than one night, which would mean there would be multiple records of that property with consecutive dates.

Next, I wanted to know how many nights each short-term property was occuppied. Since the business's goal was to evaluate pricing for rentals during 2015, the data was filtered to show only results from that year. Based on the records obtained for that year, the occupancy rate for each property was calculated as the percentage of the total number of occupied nights in a year (_Occupancy_rate_2015_).

```sql
SELECT DISTINCT 
    st_property,
    COUNT(rental_date)
FROM capstone.st_rental_dates
GROUP BY st_property;

-- FILTER THE PREVIOUS RESULTS TO OBTAIN THE NUMBER OF DAYS EACH SHORT-TERM RENTAL PROPERTY WAS OCCUPIED ONLY DURING 2015

SELECT DISTINCT
    st_property,
    COUNT(rental_date) AS number_rentedDays_2015
FROM capstone.st_rental_dates
WHERE YEAR(rental_date)=2015
GROUP BY st_property;

-- CALCULATE THE OCCUPANCY RATE DURING 2015 FOR SHORT-TERM RENTAL PROPERTIES

SELECT DISTINCT
    st_property,
    COUNT(rental_date) AS number_rentedDays_2015,
    ROUND((COUNT(rental_date)/365*100),2) AS Occupancy_rate_2015
FROM capstone.st_rental_dates
WHERE YEAR(rental_date)=2015
GROUP BY st_property
ORDER BY Occupancy_rate_2015 DESC;

```

Using the previous query to generate new data, additional information was added for each property.

Location data was added first.


```sql

-- USING THE PREVIOUS QUERY AS A SUBQUERY TO ADD THE TRANSFORMED DATA INTO THE ORIGINAL TABLES
SELECT DISTINCT
    p.st_property_id,
    p.location,
    p.property_type,
    occupancy_rate.number_rentedDays_2015,
    occupancy_rate.Occupancy_rate_2015
FROM capstone.st_property_info p 
INNER JOIN (
        SELECT DISTINCT
            rd.st_property,
            COUNT(rd.rental_date) AS Number_rentedDays_2015,
            ROUND((COUNT(rd.rental_date)/365*100),2) AS Occupancy_rate_2015
        FROM capstone.st_rental_dates AS rd
        WHERE YEAR(rd.rental_date)=2015
        GROUP BY rd.st_property
        ) AS occupancy_rate 
ON (p.st_property_id=occupancy_rate.st_property);

-- ADD DETAILED LOCATION INFO

SELECT DISTINCT
    p.st_property_id,
    p.location,
    p.property_type,
    l.city,
    l.state,
    l.zipcode,
    occupancy_rate.number_rentedDays_2015,
    occupancy_rate.Occupancy_rate_2015
FROM 
    capstone.st_property_info p JOIN location l
    JOIN (
        SELECT DISTINCT
            rd.st_property,
            COUNT(rd.rental_date) AS Number_rentedDays_2015,
            ROUND((COUNT(rd.rental_date)/365*100),2) AS Occupancy_rate_2015
        FROM capstone.st_rental_dates rd
        WHERE YEAR(rd.rental_date)=2015
        GROUP BY rd.st_property
    ) AS occupancy_rate
    ON l.location_id=p.location AND p.st_property_id=occupancy_rate.st_property;

```

Next step was to merge the generated data with the information about the properties and their rates per night. To make the code more readable in the next step, **a temporary table was created** (_st_properties_2015_).

```sql
-- ADD THE DETAILED PROPERTIES AND NIGHTLY RENTAL PRICE INFO TO THE LAST TABLE
-- Save the data in a TEMPORARY TABLE

CREATE TEMPORARY TABLE st_properties_2015
SELECT DISTINCT
    p.st_property_id,
    p.location,
    p.property_type,
    l.city,
    l.state,
    l.zipcode,
    pt.apt_house,
    pt.num_bedrooms,
    pt.kitchen,
    pt.shared,
    rp.sample_nightly_rent_price,
    occupancy_rate.number_rentedDays_2015,
    occupancy_rate.Occupancy_rate_2015
FROM 
    capstone.st_property_info p JOIN location l 
    JOIN property_type pt
    JOIN st_rental_prices rp
    JOIN (
        SELECT DISTINCT
            rd.st_property,
            COUNT(rd.rental_date) AS Number_rentedDays_2015,
            ROUND((COUNT(rd.rental_date)/365*100),2) AS Occupancy_rate_2015
        FROM capstone.st_rental_dates rd
        WHERE YEAR(rd.rental_date)=2015
        GROUP BY rd.st_property
    ) AS occupancy_rate
    ON l.location_id=p.location 
        AND p.st_property_id=occupancy_rate.st_property
        AND rp.location=p.location AND rp.property_type=p.property_type
        AND p.property_type=pt.property_type_id;

```
Followed by the joining of tables containing the summarized data for the short term properties with the table cointaining data for the Watershed long-term rental properties that will be analyzed. The **primary keys** used to merge both tables were based on the property type (which is defined by each property's characteristics) and the location (rental costs depend on those two factors, among others, so they had to match for a better comparison).

```sql
#-- COMBINE THE UNIFIED SHORT-TERM PROPERTIES DATA WITH THE WATERSHED PROPERTIES (USING AN INNER JOIN)

SELECT DISTINCT
    w.ws_property_id,
    w.location,
    w.property_type,
    w.current_monthly_rent,
    st_properties_2015.city,
    st_properties_2015.state,
    st_properties_2015.zipcode,
    st_properties_2015.apt_house,
    st_properties_2015.num_bedrooms,
    st_properties_2015.kitchen,
    st_properties_2015.shared,
    st_properties_2015.sample_nightly_rent_price AS nightly_rent_price,
    st_properties_2015.number_rentedDays_2015,
    st_properties_2015.Occupancy_rate_2015
FROM watershed_property_info w JOIN st_properties_2015
        ON w.location=st_properties_2015.location 
        AND w.property_type=st_properties_2015.property_type;

```

With this last step, the data was finally ready to be extracted and loaded into an analysis software. For this project, the data continued to be transformed and analyzed in Microsoft Excel and tableau Public.
