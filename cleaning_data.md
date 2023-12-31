# What To Fix!

## 'analytics' table:

Our largest table, containing 4,301,122 rows, as evidenced with the below query:

```
SELECT
	*
FROM
	analytics;
```
This one proved to be quite messy - present were several columns containing most, if not all, NULL values, two time colums, 'visitstarttime' and 'timeonsite' were in formats that appeared to be seconds so they will require transformation and column 'unit_price' needed to be divided by 1,000,000 to attain its true price values. The below does a unique and NULL count for all columns.

In regards to identifying the primary key, initially it was between the 'visitid' and 'fullvisitorid' however, upon further examination and with results of the below query, visitid seems like the only reasonable id as each visit to the site is represented by its own id where each visitor will retain their id, no matter how many times they return.

The issues we will be fixing will addressed through creating a view, and will be answersing questions based on. The fixes that will take place will be:

  - 'visitstarttime' will be converted from seconds to a timestamp (YYYY-MM-DD HH-MM-SS)
  - 'unit_price' needs to be divided by 1,000,000 to get true value
  - 'revenue' column has 4,285,767 / 4,301,122 rows as NULL - will be dropped
  - 'bounces' column has 3,826,283 / 4,301,122 rows as NULl - will be dropped
  - units_sold column has 4,205,975 / 4,301,122 rows as NULL - will be dropped
  - userid column has 4,301,122 / 4,301,122 (ALL) rows as NULL - will be dropped
  - timeonsite converted to timestamp (HH-MM-SS) from seconds

We can achive the above with the following created view, saving our query so that we can call upon it when answering questions later:

```
CREATE VIEW cleaned_analytics AS
SELECT
    visitnumber,
    visitid,
    TO_TIMESTAMP(visitstarttime::bigint) AS visit_time,
    date,
    fullvisitorid,
    channelgrouping,
    socialengagementtype,
    pageviews,
    (timeonsite || ' Seconds')::interval AS timeonsite_clean,
    CASE 
        WHEN unit_price IS NOT NULL THEN unit_price / 1000000
        ELSE NULL
    END AS cleaned_unit_price
FROM analytics;
```

## 'all_sessions' table:

Much like the analytics table, the all_sessions table will be cleaned as well, using a view to store the query and run it later one. The following fixes will be made:

  - "productprice" needs to be divided by 1,000,000
  - "productvariant" has 15,094 instances of "(not set)" - will be dropped
  - "time" column needs to be converted from seconds to HH:MM:SS
  - "country" has 24 columns as "(not set)" so we'll filter those out for the view
  - "city" column needs to be handled for missing entries like: 'Not available in demo 
    dataset' and '(not set)' - will replace those values with their respective countries.
  - "ecommerceaction_option" has 15,103 / 15,134 rows as NULL -- will be dropped
  - "transactionid" has 15,125 / 15,134 rows as NULL - will be dropped  
  - "transactionrevenue" has 15,130 rows as NULL - will be dropped
  - "itemrevenue" has 15,134 / 15,134 (ALL) rows as NULL - will be dropped
  - "itemquantity" has 15,134 / 15,134 (ALL) rows as NULL - will be dropped
  - "totaltransactionrevenue" has 15,053 / 15,134 rows as NULL - will be dropped
  - "sessionqualitydim" has 13,906 / 15,134 rows as NULL - will be dropped
  - "productquantity" has 15,081 / 15,134 rows as NULL - will be dropped
  - "productrevenue" has 15,130 / 15,134 rows as NULL - will be dropped
  - "searchkeyword" has 15,134 / 15,134 (ALL) rows as NULL - will be dropped
  - "transactions" has 15,053 / 15,134 rows as NULL - will be dropped
  - "productrefundamount" has 15,134 / 15,135 rows as NULL - will be dropped
  - v2productcategory is currently in a format like XXXX/XXXX/productname - while you could seperate the entire string by the '/' into seperate columns,
    we felt it would be A) give better insight as to the type of product and, B) too complicated as some rows have 2 '/' while others have 4 '/'.

We can achieve the above by creating the following view:

```
CREATE VIEW cleaned_all_sessions AS
SELECT
    fullvisitorid,
    channelgrouping,
    (time || ' seconds')::interval AS time_formatted,
    country,
	productsku,
    CASE 
        WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN country
        ELSE city
    END AS cleaned_city,
    v2productname,
   	v2productcategory,
    CASE 
        WHEN productprice IS NOT NULL THEN productprice / 1000000
        ELSE NULL
    END AS cleaned_productprice,
    currencycode,
    itemquantity,
    (timeonsite || ' Seconds')::interval AS timeonsite_formatted,
    pageviews,
    date,
    visitid,
    type
FROM 
	all_sessions;
WHERE
	country != '(not set)';
```
