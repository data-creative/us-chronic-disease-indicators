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


```` sql
SELECT i.category, count(*) AS row_count
FROM indicators i
GROUP BY i.category
ORDER BY 1
````

=>

category	| row_count
--- | ---
Alcohol	| 2744
Arthritis	| 2640
Asthma	| 4162
Cancer	| 3411
Cardiovascular Disease	| 9302
Chronic Kidney Disease	| 817
Chronic Obstructive Pulmonary Disease	| 9192
Diabetes	| 7728
Disability	| 55
Immunization	| 330
Mental Health	| 440
Nutrition, physical activity, and weight status	| 4504
Older Adults	| 1186
Oral Health	| 1797
Overarching Conditions	| 2741
Reproductive Health	| 165
Tobacco	| 2255

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

`indicator` describes `indicatorid` and vice-versa.

`locationabbr` and `locationdesc` describe `locationid` and vice-versa.
