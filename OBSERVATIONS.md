# Observations

[Download](https://chronicdata.cdc.gov/api/views/n6bi-qt4w/rows.csv?accessType=DOWNLOAD) the data from data.gov and import it into a new relational database table called `indicators`.

```` sql
SELECT count(*) FROM indicators; -- 53,469
````

There is no existing primary key :disappointed:.

```` sql
SELECT i.year, count(*) AS row_count
FROM indicators i
GROUP BY i.year
ORDER BY 1
````

=>

year	| row_count
--- | ---
2001	| 104
2006-2010	| 1323
2007	| 55
2007-2011	| 1704
2008	| 110
2009	| 279
2009-2011	| 275
2010	| 21293
2011	| 538
2011-2012	| 312
2012	| 3282
2013	| 24194

Years are not integers. Some values represent ranges. This is a categorical field, not numeric.


```` sql
SELECT
  i.category
  ,count(DISTINCT i.indicatorid) AS indicator_count
  ,count(*) AS row_count
FROM indicators i
GROUP BY 1
````

=>

category	|	indicator_count	|	row_count
--- | --- | ---
Alcohol	|	16	|	2744
Arthritis	|	8	|	2640
Asthma	|	9	|	4162
Cancer	|	20	|	3411
Cardiovascular Disease	|	18	|	9302
Chronic Kidney Disease	|	4	|	817
Chronic Obstructive Pulmonary Disease	|	16	|	9192
Diabetes	|	20	|	7728
Disability	|	1	|	55
Immunization	|	1	|	330
Mental Health	|	3	|	440
Nutrition, physical activity, and weight status	|	37	|	4504
Older Adults	|	4	|	1186
Oral Health	|	9	|	1797
Overarching Conditions	|	16	|	2741
Reproductive Health	|	3	|	165
Tobacco	|	16	|	2255

```` sql
SELECT
  i.category
  ,count(DISTINCT i.indicatorid) AS indicator_count
  ,group_concat(DISTINCT i.indicatorid  ORDER BY i.indicatorid SEPARATOR ", ") AS indicators
FROM indicators i
GROUP BY 1
````

=>



There is no data dictionary :disappointed:. Trying to make sense of the data...

What's the deal with `datavalue` vs `datavaluealt` and do I need to care about both?

```` sql
SELECT * FROM indicators i WHERE i.datavalue <> i.datavaluealt
````

It looks like alt values either have different decimal formatting, or different non-null values altogether.

What's up with `DataValueFootnoteSymbol` and other columns?

```` sql
SELECT group_concat(DISTINCT DataValueFootnoteSymbol) FROM indicators
-- ,¯,^^^,*,****,**,§§,^^,***,~,~~,~~~,~~~~,†††,^^^^,#,*, #,§§§§,‡‡
````

```` sql
SELECT group_concat(DISTINCT DataValueUnit) FROM indicators
-- %,,cases per 100,000,gallons,$,cases per 10,000,cases per million,per 100,000,Number,Average Annual Number,cases per 1,000,000,cases per 1,000,per 100...
````

```` sql
SELECT
  DataValueFootnote
  ,count(*) AS row_count
FROM indicators
GROUP BY DataValueFootnote
ORDER BY 2 desc
````

=>

DataValueFootnote	|	row_count
--- | ---
	|	43743
No data available	|	6446
Data not shown because of too few respondents or cases	|	1000
CI was calculated using Poisson variable	|	592
Sample size of denominator and/or age group for age-standardization is less than 50 or relative standard error is more than 30%	|	561
50 States + DC: US Median	|	521
NSCH Data provided by Data Resource Center for Child and Adolescent Health.  Http://www.childhealthdata.org	|	306
US Median based on the number of participating states	|	94
A confidence interval for this location is not available because a census was used instead of a sample for their surveys	|	55
Data are not available because cancer registry did not meet the data quality criteria for all the years from 2007-2011 for publication in United States Cancer statistics (USCS	|	48
State directly controlled the sale of (beer/wine/distilled spirits) at the retail and/or wholesale levels. State prices for (beer/wine/distilled spirits) combined both markups	|	30
Data are not available because of state legislation and regulations which prohibit the release of county level data to outside entities.	|	24
Data are from cancer registries that meet the data quality criteria for all invasive cancer sites combined, for all years from 2007-2011, covering approximately 99% of the U.S	|	24
Population served by CWS exceeded the US Census state population estimate; number of people was reduced by the ratio of the population estimate to the CWS population estimate	|	9
50 states + DC:  Median Percent; NSCH Data provided by Data Resource Center for Child and Adolescent Health.  http://www.childhealthdata.org	|	6
Percentage of states with applicable childcare regulations	|	5
Number of states + DC with applicable tobacco control policies	|	3
DC was not rated for this measure because the measure addresses state versus local authority and does not apply to the District of Columbia	|	1
US per capita alcohol consumption was calculated using (1) alcohol beverage sales data, either collected directly from states or provided by beverage industry sources (numerat	|	1

```` sql
SELECT group_concat(DISTINCT DataValueType) FROM indicators
````

=>

DataValueType	|	row_count
--- | ---
Crude Prevalence	|	14242
Age-adjusted Prevalence	|	11312
Number	|	7564
Crude rate	|	7098
Age-adjusted rate	|	6929
Average Annual Age-adjusted Rate	|	1009
Average Annual Crude Rate	|	1009
Average Annual Number	|	1009
Age-adjusted Mean	|	825
Percent	|	660
Crude Mean	|	495
Yes/No	|	440
Mean	|	330
US Dollars	|	165
Median	|	110
Local control of the regulation of alcohol outlet density	|	55
Mean Score	|	55
Per capita alcohol consumption	|	55
Commercial host (dram shop) liability status for alcohol service	|	55
Prevalence	|	52


Ok so there are a bunch of metric formatting and footnote concerns that will likely need to translate into the final visualization.

What's the deal with the row structure of this dataset?

```` sql
SELECT
  indicatorid
  ,Gender
  ,stratificationid1
  ,count(DISTINCT locationid) AS location_count
  ,count(*) AS row_count
FROM indicators
GROUP BY indicatorid, Gender, stratificationid1
ORDER BY indicatorid, Gender, stratificationid1
-- 406 rows
````

`Gender` describes `stratificationid1` and vice-versa.

Each combination of indicator and gender has between 55 and 100 ish rows.

```` sql
SELECT
  i.locationid
  ,i.locationabbr
  ,i.category
  ,i.indicatorid
  ,i.indicator
  ,i.datasource
  ,i.datavalueunit
  ,i.datavaluetype
  ,i.datavalue
  ,i.datavaluealt
  ,i.gender
  ,i.stratificationid1
  ,i.lowconfidenceinterval
  ,i.highconfidenceinterval
FROM indicators i
WHERE i.year = "2013" AND i.locationabbr = "CT"
````

`locationabbr` and `locationdesc` describe `locationid` and vice-versa.

`indicator` describes `indicatorid` and vice-versa.

`datavaluetype` and `datavalueunit` describe `indicator`.

```` sql
SELECT
  i.year
  ,i.locationid
  ,i.indicatorid
  ,i.stratificationid1
 ,count(DISTINCT i.datasource) AS datasource_count -- none greater than 1
 ,count(DISTINCT i.datavaluetype) AS data_value_type_count -- several greater than 1
 ,count(DISTINCT i.datavalueunit) AS data_value_unit_count -- several greater than 1
FROM indicators i
GROUP BY 1,2,3,4
````

`Year`, `locationid`, `indicatorid`, and `stratificationid1` are insufficient to form a composite primary key.


```` sql
SELECT
  i.year
  ,i.locationid
  ,i.indicatorid
  ,i.stratificationid1
  ,i.datavalueunit
  ,i.datavaluetype
 ,count(DISTINCT i.datasource) AS datasource_count -- none greater than 1
 ,count(DISTINCT i.datavalue) AS data_value_count -- none greater than 1
FROM indicators i
GROUP BY 1,2,3,4,5,6
-- HAVING data_value_count > 1
````

`Year`, `locationid`, `indicatorid`, `stratificationid1`, `datavalueunit`, and `datavaluetype` may be sufficient to form a composite primary key.

```` sql
SELECT
  i.year
  ,i.locationid
  ,i.indicatorid
  -- ,i.datasource
  ,i.datavalueunit
  ,i.datavaluetype
  ,i.stratificationid1
  ,i.datavalue
  ,i.lowconfidenceinterval
  ,i.highconfidenceinterval
FROM indicators i
WHERE i.year = "2013" AND i.locationabbr = "CT" AND i.category = "Mental Health"
ORDER BY 1,2,3,6
````

=>

year	|	locationid	|	indicatorid	|	datavalueunit	|	datavaluetype	|	stratificationid1	|	datavalue	|	lowconfidenceinterval	|	highconfidenceinterval
---	|	---	|	---	|	---	|	---	|	---	|	---	|	---	|	---
2013	|	09	|	MTH1_0	|	Number	|	Age-adjusted Mean	|	GENF	|	4.1	|	3.7	|	4.5
2013	|	09	|	MTH1_0	|	Number	|	Crude Mean	|	GENF	|	4	|	3.6	|	4.3
2013	|	09	|	MTH1_0	|	Number	|	Age-adjusted Mean	|	GENM	|	2.9	|	2.5	|	3.2
2013	|	09	|	MTH1_0	|	Number	|	Crude Mean	|	GENM	|	2.9	|	2.6	|	3.3
2013	|	09	|	MTH1_0	|	Number	|	Age-adjusted Mean	|	GENT	|	3.5	|	3.3	|	3.8
2013	|	09	|	MTH1_0	|	Number	|	Crude Mean	|	GENT	|	3.5	|	3.2	|	3.7
2013	|	09	|	MTH2_0	|	%	|	Crude Prevalence	|	GENF	|	13.5	|	11.2	|	16.3

There can be many `datavaluetype` per `indicatorid` but `datavalueunit` looks like it depends on/describes `indicatorid`. Right?

```` sql
SELECT
  i.year
  ,i.locationid
  ,i.indicatorid
  ,i.stratificationid1
  ,i.datavaluetype
  ,count(DISTINCT i.datavalueunit) AS data_value_unit_count
 ,count(DISTINCT i.datasource) AS datasource_count
 ,count(DISTINCT i.datavalue) AS data_value_count
FROM indicators i
GROUP BY 1,2,3,4,5
HAVING data_value_count > 1 OR datasource_count > 1 OR data_value_count > 1 -- zero rows
````

Indeed, this dataset row structure is a row per:
 + year
 + location
 + indicator
 + gender/strat
 + data value type

If a user chooses one value for each, comparisons can be made between values.

For indicators of the "Mental Health" category, what kind of consistency exists across metrics across years?

```` sql
SELECT
 i.year
 ,i.indicatorid
 ,count(DISTINCT i.locationid) AS location_count
FROM indicators i
WHERE i.category = "Mental Health"
GROUP BY 1,2
````

=>

year	|	indicatorid	|	location_count
--- | --- | ---
2009-2011	|	MTH3_0	|	55
2013	|	MTH1_0	|	55
2013	|	MTH2_0	|	55

There is no consistency of the same metric being measured across multiple years.

This means user selections of indicator and year are not independent, and should be limited/restricted to avoid confusion.

The first iteration of this visualization should be based off the following data filters:

```` sql
SELECT
  i.year
  ,i.locationdesc
  ,i.category
  ,i.indicator
 ,i.datavalueunit
 ,i.datavalue
 ,i.datavaluefootnote
 ,i.gender
 ,i.stratificationid1
 ,i.indicatorid
 ,i.lowconfidenceinterval
 ,i.highconfidenceinterval
FROM indicators i
WHERE i.category = "Mental Health"
  AND i.indicatorid = "MTH1_0"
  AND i.datavaluetype = "Age-adjusted Mean"
  AND i.year = "2013"
ORDER BY i.indicatorid, i.gender
````

Methodology note: [@NedMccague](https://twitter.com/NedMccague) recommends using the "Age-adjusted Mean" when comparing values across states.
