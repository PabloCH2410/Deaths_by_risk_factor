# Deaths_by_risk_factor
In this project we will use SQL and Pandas to analyse a dataset on the number of deaths by risk factor. In addition, due to the high dimensionality of the data we will use the PCA to extract insights with Python and R.

Throughout this project we will use SQL for exploratory data analysis. We will also use Pandas to do an EDA but more on the visualisation side. Before moving on to the Principal Component Analysis (PCA) part, we will look at some python visualisation that allows us to approximate the visualisation of a high dimensional dataset. Finally, we will use a PCA to understand our dataset using Sklearn and R's Tidyverse.

This can be found on [Muhammad Umair A.B.'s](https://www.kaggle.com/datasets/muhammadumairab/number-of-deaths-by-risk-factor) Kaggle page.

# Contents

- [Libraries](#libraries)
- [SQL for Exploratory Data Analysis](#sql-for-exploratory-data-analysis)
- [Exploratory Data Analysis with Python](#exploratory-data-analysis-with-python)
- [Principal Component Analysis](#principal-component-analysis)
- [References](#references)

# Libraries

## Data manipulation
```python
import pandas as pd
import numpy as np
from collections import Counter
import sqlite3
import pingouin as pg
```
## PCA modeling
```python
from sklearn.decomposition import PCA
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from yellowbrick.features import pca_decomposition
```
## Graphs
```python
import matplotlib.pyplot as plt
from matplotlib import style
import seaborn as sns
```
## Graphs configuration
```python
plt.rcParams['image.cmap'] = "bwr" #change edfault colorbar
style.use('ggplot') or plt.style.use('ggplot') #graphs style: ggplot
```

# Import dataset
```python
deaths_risk = pd.read_csv("Number of Deaths by Risk Factors.csv")

deaths_risk["Total Deaths"] = deaths_risk[deaths_risk.columns[3:]].sum(axis = 1)
```
```python
deaths_risk.head()

        Entity Code  ...  Iron deficiency  Total Deaths
0  Afghanistan  AFG  ...              564        212147
1  Afghanistan  AFG  ...              611        219979
2  Afghanistan  AFG  ...              700        237528
3  Afghanistan  AFG  ...              773        260131
4  Afghanistan  AFG  ...              812        272417

[5 rows x 32 columns]


deaths_risk.columns

Index(['Entity', 'Code', 'Year', 'Outdoor air pollution',
       'High systolic blood pressure', 'Diet high in sodium',
       'Diet low in whole grains', 'Alcohol use', 'Diet low in fruits',
       'Unsafe water source', 'Secondhand smoke', 'Low birth weight',
       'Child wasting', 'Unsafe sex', 'Diet low in nuts and seeds',
       'Household air pollution from solid fuels', 'Diet low in vegetables',
       'Low physical activity', 'Smoking', 'High fasting plasma glucose',
       'Air pollution', 'High body-mass index', 'Unsafe sanitation',
       'No access to handwashing facility',
       'Drug use - Sex: Both - Age: All Ages (Number)',
       'Low bone mineral density', 'Vitamin A deficiency', 'Child stunting',
       'Discontinued breastfeeding', 'Non-exclusive breastfeeding',
       'Iron deficiency', 'Total Deaths'],
      dtype='object')
```

# SQL for Exploratory Data Analysis

We are going to connect to an in-memory SQLite database. This creates a connection to an in-memory SQLite database. An in-memory database is not stored on disk, but resides in RAM and is lost when the connection is closed.
```python
conn = sqlite3.connect(':memory:')

#Convert the dataframe to SQLite table
deaths_risk.to_sql('deaths_risk', conn, index=False, if_exists='replace')
```
First 5 rows
```python
query = "SELECT * FROM deaths_risk LIMIT 5;"
print(pd.read_sql_query(query, conn))

        Entity Code  ...  Iron deficiency  Total Deaths
0  Afghanistan  AFG  ...              564        212147
1  Afghanistan  AFG  ...              611        219979
2  Afghanistan  AFG  ...              700        237528
3  Afghanistan  AFG  ...              773        260131
4  Afghanistan  AFG  ...              812        272417
```
Total number of rows
```python
query = "SELECT COUNT(*) AS total_records FROM deaths_risk;"
print(pd.read_sql_query(query, conn))

   total_records
0           6840
```
Range of years
```python
query = """
SELECT MIN(Year) AS min_year, MAX(Year) AS max_year 
FROM deaths_risk;
"""
print(pd.read_sql_query(query, conn))

   min_year  max_year
0      1990      2019
```
Number of records per country. That is, for each country, 30 years of number of deaths per risk factor have been recorded.
```python
query = """
SELECT Entity, COUNT(*) AS total_records
FROM deaths_risk
GROUP BY Entity
ORDER BY total_records DESC;
"""
print(pd.read_sql_query(query, conn))

                             Entity  total_records
0                          Zimbabwe             30
1                            Zambia             30
2                             Yemen             30
3    World Bank Upper Middle Income             30
4    World Bank Lower Middle Income             30
..                              ...            ...
223                  American Samoa             30
224                         Algeria             30
225                         Albania             30
226            African Region (WHO)             30
227                     Afghanistan             30

[228 rows x 2 columns]
```
Total deaths by cause in all years. 
In this case we use UNION ALL which allows us to join queries (it is like appending queries) but without removing duplicates. With this we get that, for each cause of death we get the total of these for the 30 years analysed for each country. Finally, in the last part of the query we specify that the results are ordered from highest to lowest. In addition, in the pimea quey we specify that the aco of ieso be called Cause and the calculated allo total_deaths_millions. We divide each value in each query by 1000000.0 to express the result in millions.
```python
query = """
SELECT 'Outdoor air pollution' AS cause, SUM("Outdoor air pollution") / 1000000.0 AS total_deaths_millions FROM deaths_risk
UNION ALL
SELECT 'High systolic blood pressure', SUM("High systolic blood pressure") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet high in sodium', SUM("Diet high in sodium") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in whole grains', SUM("Diet low in whole grains") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Alcohol use', SUM("Alcohol use") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in fruits', SUM("Diet low in fruits") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe water source', SUM("Unsafe water source") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Secondhand smoke', SUM("Secondhand smoke") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low birth weight', SUM("Low birth weight") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Child wasting', SUM("Child wasting") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe sex', SUM("Unsafe sex") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in nuts and seeds', SUM("Diet low in nuts and seeds") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Household air pollution from solid fuels', SUM("Household air pollution from solid fuels") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in vegetables', SUM("Diet low in vegetables") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low physical activity', SUM("Low physical activity") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Smoking', SUM("Smoking") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'High fasting plasma glucose', SUM("High fasting plasma glucose") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Air pollution', SUM("Air pollution") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'High body-mass index', SUM("High body-mass index") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe sanitation', SUM("Unsafe sanitation") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'No access to handwashing facility', SUM("No access to handwashing facility") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Drug use - Sex: Both - Age: All Ages (Number)', SUM("Drug use - Sex: Both - Age: All Ages (Number)") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low bone mineral density', SUM("Low bone mineral density") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Vitamin A deficiency', SUM("Vitamin A deficiency") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Child stunting', SUM("Child stunting") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Discontinued breastfeeding', SUM("Discontinued breastfeeding") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Non-exclusive breastfeeding', SUM("Non-exclusive breastfeeding") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Iron deficiency', SUM("Iron deficiency") / 1000000.0 FROM deaths_risk
ORDER BY total_deaths_millions DESC;
"""
print(pd.read_sql_query(query, conn))

                                           cause  total_deaths_millions
0                    High systolic blood pressure            1533.698055
1                                         Smoking            1244.589477
2                                   Air pollution            1126.905606
3                     High fasting plasma glucose             804.070651
4                            High body-mass index             614.710223
5                           Outdoor air pollution             578.543915
6        Household air pollution from solid fuels             572.107713
7                                Low birth weight             404.418509
8                                     Alcohol use             375.164443
9                                   Child wasting             341.482658
10                            Unsafe water source             301.550860
11                            Diet high in sodium             277.000599
12                       Diet low in whole grains             264.648439
13                              Unsafe sanitation             215.607381
14                               Secondhand smoke             207.689813
15                                     Unsafe sex             189.098436
16                             Diet low in fruits             163.871101
17              No access to handwashing facility             149.111278
18                          Low physical activity             112.785336
19                     Diet low in nuts and seeds              88.895029
20                         Diet low in vegetables              81.960007
21                                 Child stunting              76.364015
22  Drug use - Sex: Both - Age: All Ages (Number)              70.350782
23                       Low bone mineral density              55.968117
24                    Non-exclusive breastfeeding              49.055475
25                           Vitamin A deficiency              16.905706
26                                Iron deficiency               9.722127
27                     Discontinued breastfeeding               2.951164
```
Thus, we can see that the 3 main risk factors that have produced the most deaths are High systolic blood pressure, Smoking and Air pollution.
```python
query = """
SELECT 'Outdoor air pollution' AS cause, SUM("Outdoor air pollution") / 1000000.0 AS total_deaths_millions FROM deaths_risk
UNION ALL
SELECT 'High systolic blood pressure', SUM("High systolic blood pressure") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet high in sodium', SUM("Diet high in sodium") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in whole grains', SUM("Diet low in whole grains") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Alcohol use', SUM("Alcohol use") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in fruits', SUM("Diet low in fruits") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe water source', SUM("Unsafe water source") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Secondhand smoke', SUM("Secondhand smoke") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low birth weight', SUM("Low birth weight") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Child wasting', SUM("Child wasting") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe sex', SUM("Unsafe sex") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in nuts and seeds', SUM("Diet low in nuts and seeds") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Household air pollution from solid fuels', SUM("Household air pollution from solid fuels") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Diet low in vegetables', SUM("Diet low in vegetables") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low physical activity', SUM("Low physical activity") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Smoking', SUM("Smoking") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'High fasting plasma glucose', SUM("High fasting plasma glucose") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Air pollution', SUM("Air pollution") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'High body-mass index', SUM("High body-mass index") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Unsafe sanitation', SUM("Unsafe sanitation") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'No access to handwashing facility', SUM("No access to handwashing facility") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Drug use - Sex: Both - Age: All Ages (Number)', SUM("Drug use - Sex: Both - Age: All Ages (Number)") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Low bone mineral density', SUM("Low bone mineral density") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Vitamin A deficiency', SUM("Vitamin A deficiency") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Child stunting', SUM("Child stunting") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Discontinued breastfeeding', SUM("Discontinued breastfeeding") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Non-exclusive breastfeeding', SUM("Non-exclusive breastfeeding") / 1000000.0 FROM deaths_risk
UNION ALL
SELECT 'Iron deficiency', SUM("Iron deficiency") / 1000000.0 FROM deaths_risk
ORDER BY total_deaths_millions DESC
LIMIT 3;
"""
print(pd.read_sql_query(query, conn))

                          cause  total_deaths_millions
0  High systolic blood pressure            1533.698055
1                       Smoking            1244.589477
2                 Air pollution            1126.905606
```
Average deaths per risk factor over the 30 years
```python
query = """
SELECT 'Outdoor air pollution' AS cause, AVG("Outdoor air pollution") AS avg_deaths FROM deaths_risk
UNION ALL
SELECT 'High systolic blood pressure', AVG("High systolic blood pressure") FROM deaths_risk
UNION ALL
SELECT 'Diet high in sodium', AVG("Diet high in sodium") FROM deaths_risk
UNION ALL
SELECT 'Diet low in whole grains', AVG("Diet low in whole grains") FROM deaths_risk
UNION ALL
SELECT 'Alcohol use', AVG("Alcohol use") FROM deaths_risk
UNION ALL
SELECT 'Diet low in fruits', AVG("Diet low in fruits") FROM deaths_risk
UNION ALL
SELECT 'Unsafe water source', AVG("Unsafe water source") FROM deaths_risk
UNION ALL
SELECT 'Secondhand smoke', AVG("Secondhand smoke") FROM deaths_risk
UNION ALL
SELECT 'Low birth weight', AVG("Low birth weight") FROM deaths_risk
UNION ALL
SELECT 'Child wasting', AVG("Child wasting") FROM deaths_risk
UNION ALL
SELECT 'Unsafe sex', AVG("Unsafe sex") FROM deaths_risk
UNION ALL
SELECT 'Diet low in nuts and seeds', AVG("Diet low in nuts and seeds") FROM deaths_risk
UNION ALL
SELECT 'Household air pollution from solid fuels', AVG("Household air pollution from solid fuels") FROM deaths_risk
UNION ALL
SELECT 'Diet low in vegetables', AVG("Diet low in vegetables") FROM deaths_risk
UNION ALL
SELECT 'Low physical activity', AVG("Low physical activity") FROM deaths_risk
UNION ALL
SELECT 'Smoking', AVG("Smoking") FROM deaths_risk
UNION ALL
SELECT 'High fasting plasma glucose', AVG("High fasting plasma glucose") FROM deaths_risk
UNION ALL
SELECT 'Air pollution', AVG("Air pollution") FROM deaths_risk
UNION ALL
SELECT 'High body-mass index', AVG("High body-mass index") FROM deaths_risk
UNION ALL
SELECT 'Unsafe sanitation', AVG("Unsafe sanitation") FROM deaths_risk
UNION ALL
SELECT 'No access to handwashing facility', AVG("No access to handwashing facility") FROM deaths_risk
UNION ALL
SELECT 'Drug use - Sex: Both - Age: All Ages (Number)', AVG("Drug use - Sex: Both - Age: All Ages (Number)") FROM deaths_risk
UNION ALL
SELECT 'Low bone mineral density', AVG("Low bone mineral density") FROM deaths_risk
UNION ALL
SELECT 'Vitamin A deficiency', AVG("Vitamin A deficiency") FROM deaths_risk
UNION ALL
SELECT 'Child stunting', AVG("Child stunting") FROM deaths_risk
UNION ALL
SELECT 'Discontinued breastfeeding', AVG("Discontinued breastfeeding") FROM deaths_risk
UNION ALL
SELECT 'Non-exclusive breastfeeding', AVG("Non-exclusive breastfeeding") FROM deaths_risk
UNION ALL
SELECT 'Iron deficiency', AVG("Iron deficiency") FROM deaths_risk
ORDER BY avg_deaths DESC;
"""
print(pd.read_sql_query(query, conn))

                                            cause     avg_deaths
0                    High systolic blood pressure  224224.861842
1                                         Smoking  181957.525877
2                                   Air pollution  164752.281579
3                     High fasting plasma glucose  117554.188743
4                            High body-mass index   89869.915643
5                           Outdoor air pollution   84582.443713
6        Household air pollution from solid fuels   83641.478509
7                                Low birth weight   59125.513012
8                                     Alcohol use   54848.602778
9                                   Child wasting   49924.365205
10                            Unsafe water source   44086.383041
11                            Diet high in sodium   40497.163596
12                       Diet low in whole grains   38691.292251
13                              Unsafe sanitation   31521.546930
14                               Secondhand smoke   30364.007749
15                                     Unsafe sex   27645.970175
16                             Diet low in fruits   23957.763304
17              No access to handwashing facility   21799.894444
18                          Low physical activity   16489.084211
19                     Diet low in nuts and seeds   12996.349269
20                         Diet low in vegetables   11982.457164
21                                 Child stunting   11164.329678
22  Drug use - Sex: Both - Age: All Ages (Number)   10285.202047
23                       Low bone mineral density    8182.473246
24                    Non-exclusive breastfeeding    7171.853070
25                           Vitamin A deficiency    2471.594444
26                                Iron deficiency    1421.363596
27                     Discontinued breastfeeding     431.456725
```
Total deaths by year
```python
query = """
SELECT Year, 
       SUM("Outdoor air pollution") AS "Outdoor air pollution",
       SUM("High systolic blood pressure") AS "High systolic blood pressure",
       SUM("Diet high in sodium") AS "Diet high in sodium",
       SUM("Diet low in whole grains") AS "Diet low in whole grains",
       SUM("Alcohol use") AS "Alcohol use",
       SUM("Diet low in fruits") AS "Diet low in fruits",
       SUM("Unsafe water source") AS "Unsafe water source",
       SUM("Secondhand smoke") AS "Secondhand smoke",
       SUM("Low birth weight") AS "Low birth weight",
       SUM("Child wasting") AS "Child wasting",
       SUM("Unsafe sex") AS "Unsafe sex",
       SUM("Diet low in nuts and seeds") AS "Diet low in nuts and seeds",
       SUM("Household air pollution from solid fuels") AS "Household air pollution from solid fuels",
       SUM("Diet low in vegetables") AS "Diet low in vegetables",
       SUM("Low physical activity") AS "Low physical activity",
       SUM("Smoking") AS "Smoking",
       SUM("High fasting plasma glucose") AS "High fasting plasma glucose",
       SUM("Air pollution") AS "Air pollution",
       SUM("High body-mass index") AS "High body-mass index",
       SUM("Unsafe sanitation") AS "Unsafe sanitation",
       SUM("No access to handwashing facility") AS "No access to handwashing facility",
       SUM("Drug use - Sex: Both - Age: All Ages (Number)") AS "Drug use - Sex: Both - Age: All Ages (Number)",
       SUM("Low bone mineral density") AS "Low bone mineral density",
       SUM("Vitamin A deficiency") AS "Vitamin A deficiency",
       SUM("Child stunting") AS "Child stunting",
       SUM("Discontinued breastfeeding") AS "Discontinued breastfeeding",
       SUM("Non-exclusive breastfeeding") AS "Non-exclusive breastfeeding",
       SUM("Iron deficiency") AS "Iron deficiency"
FROM deaths_risk
GROUP BY Year
ORDER BY Year;
"""
print(pd.read_sql_query(query, conn))

    Year  Outdoor air pollution  ...  Non-exclusive breastfeeding  Iron deficiency
0   1990               13570227  ...                      2749122           407254
1   1991               13812025  ...                      2695526           401035
2   1992               14135364  ...                      2587429           401213
3   1993               14596561  ...                      2489041           390794
4   1994               14955533  ...                      2398769           387353
5   1995               15197778  ...                      2294594           381184
6   1996               15435956  ...                      2193951           376287
7   1997               15832893  ...                      2097617           378551
8   1998               16174040  ...                      2016555           376806
9   1999               16690632  ...                      1941699           375546
10  2000               17069510  ...                      1869131           372125
11  2001               17433358  ...                      1795584           364299
12  2002               17863655  ...                      1726216           353195
13  2003               18268267  ...                      1663787           340970
14  2004               18524193  ...                      1604006           333583
15  2005               18946050  ...                      1543235           330856
16  2006               19244585  ...                      1489777           322963
17  2007               19779512  ...                      1426880           311586
18  2008               20537588  ...                      1366491           303619
19  2009               20975157  ...                      1305985           296596
20  2010               21565323  ...                      1245882           290127
21  2011               22167226  ...                      1189328           282421
22  2012               22783931  ...                      1125926           270680
23  2013               23477363  ...                      1070262           264164
24  2014               23983180  ...                      1007502           251410
25  2015               24494406  ...                       947036           242119
26  2016               24639742  ...                       884484           235241
27  2017               24746020  ...                       834930           228822
28  2018               25345314  ...                       769656           227236
29  2019               26298526  ...                       725074           224092

[30 rows x 29 columns]
```
Most frequent causes of death globally in the last year
```python
query = '''
SELECT "Type", SUM(deaths) AS total_deaths
FROM (
    SELECT "Year",
           "Outdoor air pollution" AS deaths, 'Outdoor air pollution' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "High systolic blood pressure" AS deaths, 'High systolic blood pressure' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Diet high in sodium" AS deaths, 'Diet high in sodium' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Diet low in whole grains" AS deaths, 'Diet low in whole grains' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Alcohol use" AS deaths, 'Alcohol use' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Diet low in fruits" AS deaths, 'Diet low in fruits' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Unsafe water source" AS deaths, 'Unsafe water source' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Secondhand smoke" AS deaths, 'Secondhand smoke' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Low birth weight" AS deaths, 'Low birth weight' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Child wasting" AS deaths, 'Child wasting' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Unsafe sex" AS deaths, 'Unsafe sex' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Diet low in nuts and seeds" AS deaths, 'Diet low in nuts and seeds' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Household air pollution from solid fuels" AS deaths, 'Household air pollution from solid fuels' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Diet low in vegetables" AS deaths, 'Diet low in vegetables' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Low physical activity" AS deaths, 'Low physical activity' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Smoking" AS deaths, 'Smoking' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "High fasting plasma glucose" AS deaths, 'High fasting plasma glucose' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Air pollution" AS deaths, 'Air pollution' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "High body-mass index" AS deaths, 'High body-mass index' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Unsafe sanitation" AS deaths, 'Unsafe sanitation' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "No access to handwashing facility" AS deaths, 'No access to handwashing facility' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Drug use - Sex: Both - Age: All Ages (Number)" AS deaths, 'Drug use - Sex: Both - Age: All Ages (Number)' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Low bone mineral density" AS deaths, 'Low bone mineral density' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Vitamin A deficiency" AS deaths, 'Vitamin A deficiency' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Child stunting" AS deaths, 'Child stunting' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Discontinued breastfeeding" AS deaths, 'Discontinued breastfeeding' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Non-exclusive breastfeeding" AS deaths, 'Non-exclusive breastfeeding' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
    UNION ALL
    SELECT "Year",
           "Iron deficiency" AS deaths, 'Iron deficiency' AS Type FROM deaths_risk WHERE "Year" = (SELECT MAX("Year") FROM deaths_risk)
) AS deaths_data
GROUP BY "Type"
ORDER BY total_deaths DESC;
'''
print(pd.read_sql_query(query, conn))

                                             Type  total_deaths
0                    High systolic blood pressure      63948236
1                                         Smoking      46437199
2                     High fasting plasma glucose      38576633
3                                   Air pollution      38095802
4                            High body-mass index      29796826
5                           Outdoor air pollution      26298526
6                                     Alcohol use      14580012
7        Household air pollution from solid fuels      12666348
8                             Diet high in sodium      11159561
9                        Diet low in whole grains      10900754
10                               Low birth weight       9132954
11                               Secondhand smoke       7627556
12                            Unsafe water source       6729834
13                             Diet low in fruits       6137847
14                                     Unsafe sex       5375581
15                                  Child wasting       5205356
16                          Low physical activity       4984338
17                              Unsafe sanitation       4103489
18                     Diet low in nuts and seeds       3370930
19              No access to handwashing facility       3362423
20                         Diet low in vegetables       3060239
21  Drug use - Sex: Both - Age: All Ages (Number)       3016199
22                       Low bone mineral density       2649646
23                                 Child stunting        847722
24                    Non-exclusive breastfeeding        725074
25                                Iron deficiency        224092
26                           Vitamin A deficiency        121072
27                     Discontinued breastfeeding         40318
```
Total deaths per country
```python
query = '''
SELECT Entity, Code, SUM("Outdoor air pollution" + "High systolic blood pressure" + "Diet high in sodium" + "Diet low in whole grains" + 
             "Alcohol use" + "Diet low in fruits" + "Unsafe water source" + "Secondhand smoke" + "Low birth weight" + "Child wasting" + 
             "Unsafe sex" + "Diet low in nuts and seeds" + "Household air pollution from solid fuels" + "Diet low in vegetables" + 
             "Low physical activity" + "Smoking" + "High fasting plasma glucose" + "Air pollution" + "High body-mass index" + 
             "Unsafe sanitation" + "No access to handwashing facility" + "Drug use - Sex: Both - Age: All Ages (Number)" + 
             "Low bone mineral density" + "Vitamin A deficiency" + "Child stunting" + "Discontinued breastfeeding" + 
             "Non-exclusive breastfeeding" + "Iron deficiency") AS total_deaths
FROM deaths_risk
GROUP BY Entity, Code
ORDER BY total_deaths DESC;
'''
print(pd.read_sql_query(query, conn))

                              Entity      Code  total_deaths
0                             World  OWID_WRL    1706095617
1                               G20      None    1114305353
2    World Bank Lower Middle Income      None     681638530
3    World Bank Upper Middle Income      None     605028525
4          East Asia & Pacific (WB)      None     513644975
..                              ...       ...           ...
223                    Cook Islands       COK          4824
224                          Tuvalu       TUV          3570
225                           Nauru       NRU          2683
226                            Niue       NIU           809
227                         Tokelau       TKL           347

[228 rows x 3 columns]
```
Which country had the most deaths in the 30 years?
```python
query = '''
SELECT Entity, Code, SUM("Outdoor air pollution" + "High systolic blood pressure" + "Diet high in sodium" + "Diet low in whole grains" + 
             "Alcohol use" + "Diet low in fruits" + "Unsafe water source" + "Secondhand smoke" + "Low birth weight" + "Child wasting" + 
             "Unsafe sex" + "Diet low in nuts and seeds" + "Household air pollution from solid fuels" + "Diet low in vegetables" + 
             "Low physical activity" + "Smoking" + "High fasting plasma glucose" + "Air pollution" + "High body-mass index" + 
             "Unsafe sanitation" + "No access to handwashing facility" + "Drug use - Sex: Both - Age: All Ages (Number)" + 
             "Low bone mineral density" + "Vitamin A deficiency" + "Child stunting" + "Discontinued breastfeeding" + 
             "Non-exclusive breastfeeding" + "Iron deficiency") AS total_deaths
FROM deaths_risk
WHERE Entity NOT IN ("World", "G20", "World Bank Lower Middle Income", "World Bank Upper Middle Income", "East Asia & Pacific (WB)", 
                     "South-East Asia Region (WHO)", "Western Pacific Region (WHO)", "South Asia (WB)")
GROUP BY Entity, Code
ORDER BY total_deaths DESC
LIMIT 1;
'''
print(pd.read_sql_query(query, conn))

  Entity Code  total_deaths
0  China  CHN     345482042
```
Countries with the largest increase in deaths between the first and last available year
```python
query = '''
WITH deaths_per_year AS (
    SELECT Entity, Code, Year,
           SUM("Outdoor air pollution" + "High systolic blood pressure" + "Diet high in sodium" + "Diet low in whole grains" + 
               "Alcohol use" + "Diet low in fruits" + "Unsafe water source" + "Secondhand smoke" + "Low birth weight" + "Child wasting" + 
               "Unsafe sex" + "Diet low in nuts and seeds" + "Household air pollution from solid fuels" + "Diet low in vegetables" + 
               "Low physical activity" + "Smoking" + "High fasting plasma glucose" + "Air pollution" + "High body-mass index" + 
               "Unsafe sanitation" + "No access to handwashing facility" + "Drug use - Sex: Both - Age: All Ages (Number)" + 
               "Low bone mineral density" + "Vitamin A deficiency" + "Child stunting" + "Discontinued breastfeeding" + 
               "Non-exclusive breastfeeding" + "Iron deficiency") AS total_deaths
    FROM deaths_risk
    GROUP BY Entity, Code, Year
),
first_year_deaths AS (
    SELECT Entity, Code, SUM(total_deaths) AS deaths_first_year
    FROM deaths_per_year
    WHERE Year = (SELECT MIN(Year) FROM deaths_risk)
    GROUP BY Entity, Code
),
last_year_deaths AS (
    SELECT Entity, Code, SUM(total_deaths) AS deaths_last_year
    FROM deaths_per_year
    WHERE Year = (SELECT MAX(Year) FROM deaths_risk)
    GROUP BY Entity, Code
)
SELECT f.Entity, f.Code, l.deaths_last_year - f.deaths_first_year AS death_increase
FROM first_year_deaths f
JOIN last_year_deaths l ON f.Entity = l.Entity AND f.Code = l.Code
ORDER BY death_increase DESC
LIMIT 10;
'''
print(pd.read_sql_query(query, conn))

         Entity      Code  death_increase
0          World  OWID_WRL         8822823
1          China       CHN         3355951
2          India       IND         1581975
3      Indonesia       IDN          586442
4  United States       USA          390698
5    Philippines       PHL          341963
6       Pakistan       PAK          325083
7         Mexico       MEX          294587
8        Vietnam       VNM          269464
9          Egypt       EGY          228821
```
And this same evolution expressed in %: we observe that for example China is the country with the highest increase in deaths in absolute terms but the United Arab Emirates is the country with the fastest increase in deaths taking into account all risk factors between 2019 and 1990.
```python
query = '''
WITH deaths_per_year AS (
    SELECT Entity, Code, Year,
           SUM("Outdoor air pollution" + "High systolic blood pressure" + "Diet high in sodium" + "Diet low in whole grains" + 
               "Alcohol use" + "Diet low in fruits" + "Unsafe water source" + "Secondhand smoke" + "Low birth weight" + "Child wasting" + 
               "Unsafe sex" + "Diet low in nuts and seeds" + "Household air pollution from solid fuels" + "Diet low in vegetables" + 
               "Low physical activity" + "Smoking" + "High fasting plasma glucose" + "Air pollution" + "High body-mass index" + 
               "Unsafe sanitation" + "No access to handwashing facility" + "Drug use - Sex: Both - Age: All Ages (Number)" + 
               "Low bone mineral density" + "Vitamin A deficiency" + "Child stunting" + "Discontinued breastfeeding" + 
               "Non-exclusive breastfeeding" + "Iron deficiency") AS total_deaths
    FROM deaths_risk
    GROUP BY Entity, Code, Year
),
first_year_deaths AS (
    SELECT Entity, Code, SUM(total_deaths) AS deaths_first_year
    FROM deaths_per_year
    WHERE Year = (SELECT MIN(Year) FROM deaths_risk)
    GROUP BY Entity, Code
),
last_year_deaths AS (
    SELECT Entity, Code, SUM(total_deaths) AS deaths_last_year
    FROM deaths_per_year
    WHERE Year = (SELECT MAX(Year) FROM deaths_risk)
    GROUP BY Entity, Code
)
SELECT f.Entity, f.Code, 
       ((l.deaths_last_year - f.deaths_first_year) * 100.0 / f.deaths_first_year) AS death_increase_percent
FROM first_year_deaths f
JOIN last_year_deaths l ON f.Entity = l.Entity AND f.Code = l.Code
ORDER BY death_increase_percent DESC
LIMIT 10;
'''
print(pd.read_sql_query(query, conn))

                        Entity Code  death_increase_percent
0          United Arab Emirates  ARE              516.358839
1                         Qatar  QAT              245.473833
2                        Kuwait  KWT              173.653654
3                        Jordan  JOR              151.471861
4      Northern Mariana Islands  MNP              142.774566
5                       Vanuatu  VUT              142.758621
6                          Guam  GUM              130.769231
7                       Bahrain  BHR              125.386064
8  United States Virgin Islands  VIR              122.872340
9                       Andorra  AND              121.327014
```
Top three causes of death by year

ROW_NUMBER() OVER (PARTITION BY Year ORDER BY total_deaths DESC) AS rank: Assigns a row number to each cause within each year based on the total deaths (the cause with the highest number of deaths gets number 1). The next line uses the ROW_NUMBER() window function to assign a sequential row number to each row, within each data partition based on the year. The partitioning is done by the ‘Year’ column, which means that the data is grouped by year. Within each year group, the rows are sorted in descending order according to the total number of deaths (‘total_deaths’). This allows a ‘rank’ to be assigned to each cause of death within a year, where the value 1 is assigned to the cause with the most deaths, 2 to the second highest, and so on. The result is stored in a new column called ‘rank’, which indicates the position of each cause of death within its respective year in terms of total number of deaths.

MAX(CASE WHEN rank = 1 THEN cause END) AS Top1_Cause: Uses a CASE expression to select the cause of death with rank 1 (highest) for each year.

MAX(CASE WHEN rank = 1 THEN total_deaths / 1000000.0 END) AS Top1_Deaths_Millions: Similar to above, but also converts deaths to millions.

We see that the leading cause of death from 1990 to 2019 was High systolic blood pressure, while the second leading cause in 1990 was Air pollution and in 2019 it was Smoking, and the third leading cause in 1990 was Smoking and in 2019 it was High fasting plasma glucose.
```python
query = """
WITH CauseRank AS (
    SELECT 
        Year,
        cause,
        total_deaths,
        ROW_NUMBER() OVER (PARTITION BY Year ORDER BY total_deaths DESC) AS rank
    FROM (
        SELECT Year, 'Outdoor air pollution' AS cause, SUM("Outdoor air pollution") AS total_deaths FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'High systolic blood pressure', SUM("High systolic blood pressure") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Diet high in sodium', SUM("Diet high in sodium") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Diet low in whole grains', SUM("Diet low in whole grains") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Alcohol use', SUM("Alcohol use") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Diet low in fruits', SUM("Diet low in fruits") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Unsafe water source', SUM("Unsafe water source") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Secondhand smoke', SUM("Secondhand smoke") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Low birth weight', SUM("Low birth weight") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Child wasting', SUM("Child wasting") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Unsafe sex', SUM("Unsafe sex") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Diet low in nuts and seeds', SUM("Diet low in nuts and seeds") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Household air pollution from solid fuels', SUM("Household air pollution from solid fuels") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Diet low in vegetables', SUM("Diet low in vegetables") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Low physical activity', SUM("Low physical activity") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Smoking', SUM("Smoking") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'High fasting plasma glucose', SUM("High fasting plasma glucose") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Air pollution', SUM("Air pollution") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'High body-mass index', SUM("High body-mass index") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Unsafe sanitation', SUM("Unsafe sanitation") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'No access to handwashing facility', SUM("No access to handwashing facility") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Drug use - Sex: Both - Age: All Ages (Number)', SUM("Drug use - Sex: Both - Age: All Ages (Number)") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Low bone mineral density', SUM("Low bone mineral density") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Vitamin A deficiency', SUM("Vitamin A deficiency") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Child stunting', SUM("Child stunting") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Discontinued breastfeeding', SUM("Discontinued breastfeeding") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Non-exclusive breastfeeding', SUM("Non-exclusive breastfeeding") FROM deaths_risk GROUP BY Year
        UNION ALL
        SELECT Year, 'Iron deficiency', SUM("Iron deficiency") FROM deaths_risk GROUP BY Year
    ) AS CauseTotals
)
SELECT 
    Year,
    MAX(CASE WHEN rank = 1 THEN cause END) AS Top1_Cause,
    MAX(CASE WHEN rank = 1 THEN total_deaths / 1000000.0 END) AS Top1_Deaths_Millions,
    MAX(CASE WHEN rank = 2 THEN cause END) AS Top2_Cause,
    MAX(CASE WHEN rank = 2 THEN total_deaths / 1000000.0 END) AS Top2_Deaths_Millions,
    MAX(CASE WHEN rank = 3 THEN cause END) AS Top3_Cause,
    MAX(CASE WHEN rank = 3 THEN total_deaths / 1000000.0 END) AS Top3_Deaths_Millions
FROM CauseRank
GROUP BY Year
ORDER BY Year;
"""
df_top_causes_per_year = pd.read_sql_query(query, conn)

# Renombrar las columnas para mayor claridad
df_top_causes_per_year.columns = [
    'Year', 
    'Top 1 Cause', 
    'Top 1 Deaths (Millions)', 
    'Top 2 Cause', 
    'Top 2 Deaths (Millions)', 
    'Top 3 Cause', 
    'Top 3 Deaths (Millions)']

print(df_top_causes_per_year)
```
![df_top_causes_per_year (1)](https://github.com/user-attachments/assets/f9a617c1-b9fd-4643-a933-5f2976d33add)
![df_top_causes_per_year (2)](https://github.com/user-attachments/assets/d1e82a18-2769-4800-a27b-9b9c754fa476)

```python
#Log off
conn.close()
```

# Exploratory Data Analysis with Python

After performing an EDA with SQL, we are going to perform an EDA but this time with Python in order to be able to perform different visualisations.
```python
np.unique(deaths_risk['Entity'].values)
Counter(np.unique(deaths_risk['Entity'].values))
len(np.unique(deaths_risk['Entity'].values)) #We have 228 countries in our dataset

np.unique(deaths_risk['Year'].values) #it seems that by every country we have dat from 1990 to 2019 (as we knew form our SQL analysis). Lets check it with a few filters

array([1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000,
       2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011,
       2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019], dtype=int64)
```
We can see data from 1990 to 2019 in this samples. In this case, we can see that in average the number of deaths due to outdoor air pollution follows different trends in this three countries: the rate of deaths from this cause in Vietnam stands out much more than in the other two countries, with only Wales showing a negative slope
```python
vietnam_data = deaths_risk.loc[deaths_risk['Entity'] == "Vietnam"].groupby(['Entity', 'Year'])['Outdoor air pollution'].mean()
afghanistan_data = deaths_risk.loc[deaths_risk['Entity'] == "Afghanistan"].groupby(['Entity', 'Year'])['Outdoor air pollution'].mean()
wales_data = deaths_risk.loc[deaths_risk['Entity'] == "Wales"].groupby(['Entity', 'Year'])['Outdoor air pollution'].mean()

plt.figure(figsize=(10, 6))

plt.plot(vietnam_data.index.get_level_values('Year'), vietnam_data, label='Vietnam')
plt.plot(afghanistan_data.index.get_level_values('Year'), afghanistan_data, label='Afghanistan')
plt.plot(wales_data.index.get_level_values('Year'), wales_data, label='Wales')

plt.title('Outdoor Air Pollution Over Time')
plt.xlabel('Year')
plt.ylabel('Outdoor Air Pollution (Mean Deaths)')
plt.legend()
plt.show()
```
![Outdoor Air Pollution Over Time](https://github.com/user-attachments/assets/8638d3ae-555f-49cb-bacc-6ced0107d6f4)

Lets analysis numeric variables. We have 29 variables so it is quite difficult to figure out with are the most importants. Thus, in this section we are goingto analyze just a few of them in order to apply some different interesting data methods and discover some issues about it. Now, we will focus on those variables related to Diet in order to understand better the dataset
```python
death_risk_diet = deaths_risk[['Entity',
                               'Year',
                               'Diet high in sodium',
                               'Diet low in whole grains',
                               'Diet low in fruits',
                               'Diet low in nuts and seeds',
                               'Diet low in vegetables']]

europe_region_diet = death_risk_diet.loc[deaths_risk['Entity'] == "European Region (WHO)"]
```
First, lets see the evolution of deaths in Europe due to diet issues
```python
variable_cols_diet = death_risk_diet[['Diet high in sodium',
                                      'Diet low in whole grains',
                                      'Diet low in fruits',
                                      'Diet low in nuts and seeds',
                                      'Diet low in vegetables']]

fig, axes = plt.subplots(nrows=2, ncols=3, figsize=(9, 5))
axes = axes.flat

for i, colum in enumerate(variable_cols_diet):
    sns.lineplot(
        x=europe_region_diet['Year'],
        y=europe_region_diet[colum],
        ax      = axes[i]
    )
    axes[i].set_title(colum, fontsize = 7, fontweight = "bold")
    axes[i].tick_params(labelsize = 6)
    axes[i].set_xlabel("")

#We delete the empty plots
for i in [5]:
    fig.delaxes(axes[i])

fig.tight_layout()
plt.subplots_adjust(top = 0.9)
fig.suptitle('Distribution of diet variables in Europe Region')
```
![Distribution of diet variables in Europe Region](https://github.com/user-attachments/assets/154e714e-dd6b-44d1-a698-6afd57e730f8)

These risk factors follow a similar pattern, but if we want to highlight something risk factor diet low in vegetables has a more pronounced decreasing trend (without any major upward trend between 2000 and 2005 as diet low in whole grains or diet low in fruits).

After checking every variable on their separately, maybe we want to gather all them together and see the evolution across years. For this purpose, we are going to convert our DataFrame europe_region_diet with a pivot longer so we gather al the diet variables in one column and their result in otrher. With this, we can eassily pass a hue in a lineplot. Thus, we can easily see that the biggest risk factor in terms of diet is the low intake of whole grains. Moreover, a diet low in fruit and a diet low in nuts and seeds have virtually identical behaviour.
```python
europe_region_diet = pd.melt(europe_region_diet, 
                             id_vars='Year', 
                             value_vars=['Diet high in sodium',
                                         'Diet low in whole grains', 
                                         'Diet low in fruits', 
                                         'Diet low in nuts and seeds',
                                         'Diet low in vegetables'])
europe_region_diet['Entity'] = 'European Region (WHO)'

ax2 = sns.lineplot(
    x = europe_region_diet['Year'],
    y = europe_region_diet['value'],
    hue = europe_region_diet['variable'])
sns.move_legend(ax2, "upper left", bbox_to_anchor=(1, 1))
```
![sns lineplot](https://github.com/user-attachments/assets/dfd5b27c-6309-44c2-9e04-fe8495265a13)

Lets do a heatmap to see the relations between diet variables in Europe Region
```python
europe_region_diet = death_risk_diet.loc[deaths_risk['Entity'] == "European Region (WHO)"]
europe_region_diet = europe_region_diet[['Entity', 'Diet high in sodium', 'Diet low in whole grains',
       'Diet low in fruits', 'Diet low in nuts and seeds',
       'Diet low in vegetables']].drop(['Entity'], axis=1)

mask = np.triu(np.ones_like(europe_region_diet.corr()))

plt.figure(figsize=(15,10))
sns.heatmap(
    europe_region_diet.corr(method = 'spearman'),
    annot = True,
    mask = mask #to show only the lower part of the correlation matrix
    )
```
![corrmatrix](https://github.com/user-attachments/assets/7dd18bc5-a37f-4685-bcdc-406f5727ace8)
```python
#Small P-values (below 0.05): Indicate that the correlation is significant
#Large P-values: Suggest that the correlation is not significant

corr_test = pg.pairwise_corr(europe_region_diet, method='spearman')
pvals_matrix = corr_test.pivot(index='X', columns='Y', values='p-unc')
print(pvals_matrix) #all p-values < 0.05 so the correlaion between variables is significant
```
![pvals corrmatrix](https://github.com/user-attachments/assets/983b1728-c4a6-4697-a624-2bdac623b093)

Coming to this point we could use some kind visualization in order to figure out the behavior of the dimensions of the dataset. Raviz is a multivariate data visualization algorithm that plots each feature dimension uniformly around the circumference of a circle then plots points on the interior of the circle such that the point normalizes its values on the axes from the center to each arc.

The radviz graph allows us to represent multiple dimensions in a 2D plane. The interpretation of the graph is that if the points are close to a particular variable it implies that our observations have a high value on that variable. The same is true for the density of the points (observations): high density next to a variable alerts us to a high value for this variable. In this case, the visualisation matches the line graph we saw earlier, as the dots tend towards Diet low in whole grains and Diet high in sodium, the two main diet-related risk factors in Europe. Similarly, we see a tail towards Diet low in vegetables, indicating that in a few years this cause of death could be prominent.
```python
europe_region_diet = death_risk_diet.loc[deaths_risk['Entity'] == "European Region (WHO)"]

europe_region_diet = europe_region_diet.drop(['Year'], axis=1)

radviz_europeReg_diet = pd.plotting.radviz(europe_region_diet, 'Entity')
radviz_europeReg_diet.legend(bbox_to_anchor=(1.0, 1.0))
```
![radviz](https://github.com/user-attachments/assets/3618f341-9fcc-4c5d-97eb-b2d9a850fa9e)


# Principal Component Analysis

We can repeat this same script with every country and variables we want, but the truth is that it is quite difficult to figure it out the hole dataset and also where to focus, either variables or countries. We have 29 variables (29 dimensions), so we are going to perform to this dataset a dimensionality reduction in order to find some hints about this data. In this case we are going to use Principal Component Analysis (PCA), one of the most famous dimensionality reduction methods. If you are interested in PCA I really recommend [PCA with Python](https://cienciadedatos.net/documentos/py19-pca-python) by Joaquin Amat, which is the article we have followed to implement the PCA algorithm with Python.

First, we recover the original dataset. Moreover, we are going to take into account the last date available in the dataset: 2019
```python
deaths_risk = pd.read_csv("Number of Deaths by Risk Factors.csv")

deaths_risk = deaths_risk.loc[deaths_risk['Year'] == 2019]

deaths_risk.drop(deaths_risk.columns[[1,2]], 
                 axis=1, 
                 inplace=True) # Remove columns index base
```
As we did before, we can visualize all the dimensions in a RadViz
```python
radviz_deaths_risk = pd.plotting.radviz(deaths_risk, 'Entity')
radviz_deaths_risk.legend().remove()
```
![radviz 4all](https://github.com/user-attachments/assets/16a15c25-9296-4458-af87-6278a76e8152)

In order to do  the PCA we set Entity as index so we have every variable in a column
```python
deaths_risk.set_index("Entity", inplace = True) #set Entity as index
```
After this, we need to scalate. PCA identifies the variables with the highest variance. The variance of a variable is measured in its squared units, so it is necessary to standardise variables with mean 0 and standard deviation 1, otherwise the variables with the largest scale will dominate the data set.


The PCA() method from sklearn.decomposition centers the values but does not scale them. We can scale them by using StandardScaler() together with PCA() in a pipeline. The make_pipeline() method allows us to combine multiple preprocessing transformations, specifying which columns each transformation should be applied to. If we do not specify the n_components parameter in PCA(), all possible components will be calculated.
```python
#PCA model training with data scaling
pca_pipe = make_pipeline(StandardScaler(), PCA())
pca_pipe.fit(deaths_risk)

#Extraction of the trained model from the pipeline
deaths_risk_pca = pca_pipe.named_steps['pca']
```
```python
#Percentage of variance explained by each component

fig, ax = plt.subplots(figsize=(12, 8))
x_values = np.arange(1, deaths_risk_pca.n_components_ + 1)
variance_ratios = deaths_risk_pca.explained_variance_ratio_

ax.bar(x=x_values, height=variance_ratios)

for i, variance in enumerate(variance_ratios, start=1):
    ax.annotate(
        round(variance, 2),
        (i, variance),
        textcoords="offset points",
        xytext=(0, 10),
        ha='center'
    )

ax.set_xticks(x_values)
ax.set_ylim(0, 1.1)
ax.set_xlabel('PC')
ax.set_ylabel('Percentage of Variance Explained')
```
![Percentage of Variance Explained](https://github.com/user-attachments/assets/654d457e-b920-4b56-8c39-5dcfb8aa8b55)
So, we can see that with just the first principal component (PC1) we can explain the 83% of the variance of the dataset, while the PC1 along with the PC2 we can explain the 96% of the variance of the dataset

```python
cumulative_variance = deaths_risk_pca.explained_variance_ratio_.cumsum()
print(cumulative_variance)

[0.83344304 0.96469273 0.98353876 0.99240539 0.99487417 0.99657769
 0.9982281  0.99895881 0.99934392 0.99961704 0.99977405 0.99983898
 0.99989176 0.99992354 0.9999442  0.9999603  0.99996901 0.99997635
 0.99998182 0.99998677 0.99999012 0.999993   0.9999955  0.99999751
 0.99999898 0.99999953 1.         1.        ]
```
```python
#Heatmap of PCA

fig, ax = plt.subplots(figsize=(20, 15))
components = deaths_risk_pca.components_.T

im = ax.imshow(components, cmap='viridis', aspect='auto')

ax.set_yticks(np.arange(len(deaths_risk.columns)))
ax.set_yticklabels(deaths_risk.columns)

ax.set_xticks(np.arange(deaths_risk_pca.n_components_))
ax.set_xticklabels(np.arange(1, deaths_risk_pca.n_components_ + 1))

ax.grid(False)
fig.colorbar(im, ax=ax)
```
![Heatmap PCA](https://github.com/user-attachments/assets/16816c1a-c0c8-4566-984d-84cc49831e9c)

From the heatmap component we saw that all the loadings from PC1 are positive, something which we can check in these representations. For this purpose we are going to use the library Yellowbrick

## PCA plotting with Python
```python
visualize_2D = pca_decomposition(deaths_risk, 
                                 scale=True, projection = 2, 
                                 proj_features=True)
```
![visualize_2D](https://github.com/user-attachments/assets/982e0432-d48c-42b6-88e8-e3ce1c616fb5)

```python
visualizer_3D = pca_decomposition(deaths_risk, 
                                  scale=True, projection = 3)
```
![visualizer_3D](https://github.com/user-attachments/assets/3dfcc478-84fe-4eac-8d42-8540e8b56d35)


This script show us the 5 highest loadings of PC1 and PC2 (both two PC explain the 96% of the data). The loadings can be interpreted as the weight/importance of each variable in each component and, therefore, help to know what kind of information each component collects. So, with this information we can take into account the x highest loadings of a PC and explore them in any country (or a bunch of countries).

Thus, we can see that in PC1 (which explains 83% of the variance of our dataset) the variable with the greatest weight/importance is Air pollution together with Household air pollution from solid fuels and Diet low in vegetables. Therefore, we could create a subdataset with these three fields and investigate further. This is where we see the usefulness of dimensionality reduction algorithms as in this case it allows us, instead of focusing on 28 variables, to look at those that explain the most variability in our data.
```python
deaths_risk_pca_table =pd.DataFrame( 
    data    = deaths_risk_pca.components_,
    columns = deaths_risk.columns,
    index   = ['PC1', 'PC2', 'PC3', 'PC4','PC4','PC5','PC6','PC7','PC8','PC9',
               'PC10','PC11','PC12','PC13','PC14','PC15','PC16','PC17','PC18',
               'PC19','PC20','PC21','PC22','PC23','PC24','PC25','PC26','PC27'])

abs(deaths_risk_pca_table.loc['PC1']).nlargest(n=5) #we want the absolute value because the sign is the direction of the eignvector

Air pollution                               0.201133
Household air pollution from solid fuels    0.200987
Diet low in vegetables                      0.200325
Diet low in fruits                          0.199771
High fasting plasma glucose                 0.198217
Name: PC1, dtype: float64


abs(deaths_risk_pca_table.loc['PC2']).nlargest(n=5)

Vitamin A deficiency           0.323678
Child stunting                 0.287921
Non-exclusive breastfeeding    0.270637
Discontinued breastfeeding     0.266872
Child wasting                  0.254217
Name: PC2, dtype: float64
```

## PCA plotting with R

Regarding PCA, in Python we can implement the modelling and the plotting of this algorithm. But we can have more information from the last two plots (visualizer 2D and 3D). In this case, we are going to jump to R and use ggbiplot package, which will allow us to peer inside this dataset. In R we implement prcomp() to our dataset (after doing the same changes) and we obtain the same PCA results that we obtained here in Python with scikit-learn
```{r}
library(readr)
library(tidyverse)
library(ggbiplot)

Number_of_Deaths_by_Risk_Factors <- read_csv("Number of Deaths by Risk Factors.csv")

deaths_risk_R <-
  Number_of_Deaths_by_Risk_Factors %>%
  filter(Number_of_Deaths_by_Risk_Factors$Year == 2019)

pca_R <- prcomp(deaths_risk_R[,c(4:31)], 
         center = TRUE, 
         scale = TRUE)
```
The elements centre and scale stored in the pca object contain the mean and standard deviation of the variables after standardisation (on the original scale) rotation contains the value of the loadings ϕ for each component (eigenvector)
```{r}
names(pca_R)

"sdev"     "rotation" "center"   "scale"    "x"
```
```{r}
pca_R$center

Outdoor air pollution                  High systolic blood pressure 
                                  115344.4123                                   280474.7193 
                          Diet high in sodium                      Diet low in whole grains 
                                   48945.4430                                    47810.3246 
                                  Alcohol use                            Diet low in fruits 
                                   63947.4211                                    26920.3816 
                          Unsafe water source                              Secondhand smoke 
                                   29516.8158                                    33454.1930 
                             Low birth weight                                 Child wasting 
                                   40056.8158                                    22830.5088 
                                   Unsafe sex                    Diet low in nuts and seeds 
                                   23577.1096                                    14784.7807 
     Household air pollution from solid fuels                        Diet low in vegetables 
                                   55554.1579                                    13422.1009 
                        Low physical activity                                       Smoking 
                                   21861.1316                                   203671.9254 
                  High fasting plasma glucose                                 Air pollution 
                                  169195.7588                                   167086.8509 
                         High body-mass index                             Unsafe sanitation 
                                  130687.8333                                    17997.7588 
            No access to handwashing facility Drug use - Sex: Both - Age: All Ages (Number) 
                                   14747.4693                                    13228.9430 
                     Low bone mineral density                          Vitamin A deficiency 
                                   11621.2544                                      531.0175 
                               Child stunting                    Discontinued breastfeeding 
                                    3718.0789                                      176.8333 
                  Non-exclusive breastfeeding                               Iron deficiency 
                                    3180.1491                                      982.8596 
```
```{r}
pca_R$scale

Outdoor air pollution                  High systolic blood pressure 
                                  478776.3315                                  1077881.5116 
                          Diet high in sodium                      Diet low in whole grains 
                                  212609.7866                                   181282.9398 
                                  Alcohol use                            Diet low in fruits 
                                  242962.7198                                   106635.8097 
                          Unsafe water source                              Secondhand smoke 
                                  134476.9729                                   135582.9895 
                             Low birth weight                                 Child wasting 
                                  169572.5168                                    99320.2509 
                                   Unsafe sex                    Diet low in nuts and seeds 
                                   97323.6053                                    58398.1428 
     Household air pollution from solid fuels                        Diet low in vegetables 
                                  228119.7210                                    51631.9830 
                        Low physical activity                                       Smoking 
                                   82155.1988                                   809682.7889 
                  High fasting plasma glucose                                 Air pollution 
                                  634559.6526                                   675213.0917 
                         High body-mass index                             Unsafe sanitation 
                                  484918.0931                                    81657.5689 
            No access to handwashing facility Drug use - Sex: Both - Age: All Ages (Number) 
                                   64534.4530                                    50254.5079 
                     Low bone mineral density                          Vitamin A deficiency 
                                   45319.1400                                     2679.9615 
                               Child stunting                    Discontinued breastfeeding 
                                   17256.4486                                      778.2197 
                  Non-exclusive breastfeeding                               Iron deficiency 
                                   14431.2671                                     4252.7348 
```
```{r}
pca_R$rotation

                                                    PC1         PC2          PC3          PC4         PC5          PC6
Outdoor air pollution                         0.1949689 -0.14698447 -0.091214543  0.331177696 -0.05118606  0.037505437
High systolic blood pressure                  0.1958772 -0.16399331  0.089073040  0.012289499 -0.12474434 -0.011695688
Diet high in sodium                           0.1797373 -0.21496789  0.211441123  0.456550399  0.01122647  0.018054404
Diet low in whole grains                      0.1941293 -0.16774337  0.087521301 -0.154054249 -0.28488858 -0.071084074
Alcohol use                                   0.1949031 -0.15855452  0.160766173 -0.099040665  0.09977129  0.014586461
Diet low in fruits                            0.1997706 -0.12454707 -0.128646904  0.081621675  0.02480099 -0.043543995
Unsafe water source                           0.1875434  0.16423111 -0.380522432 -0.011672527  0.17060286  0.060745189
Secondhand smoke                              0.1940725 -0.16504301  0.010662743  0.283185436 -0.06414857  0.019158805
Low birth weight                              0.1915205  0.17474915 -0.218033400 -0.031040855 -0.16982321  0.022554033
Child wasting                                 0.1791389  0.25421653  0.148073523 -0.009432193 -0.06078695  0.045212433
Unsafe sex                                    0.1868309  0.15115436  0.356200914  0.051533526  0.25747764  0.528870498
Diet low in nuts and seeds                    0.1960911 -0.12376736 -0.196927033 -0.173114340 -0.33539358 -0.107220406
Household air pollution from solid fuels      0.2009868  0.06849046 -0.184083685  0.272662943  0.07257304 -0.152242952
Diet low in vegetables                        0.2003254 -0.04255523 -0.254345718 -0.261285993  0.03261864 -0.045526865
Low physical activity                         0.1913142 -0.17906208  0.129994164 -0.231468341 -0.21194518 -0.043221378
Smoking                                       0.1887658 -0.20542522  0.133595096  0.087563335  0.08983900 -0.048349754
High fasting plasma glucose                   0.1982168 -0.14323108  0.006627988 -0.156662611  0.02526181  0.002402341
Air pollution                                 0.2011334 -0.07948420 -0.110948838  0.321065351 -0.01912804 -0.022253752
High body-mass index                          0.1943265 -0.15594204  0.123232733 -0.275282733 -0.13130916  0.038200430
Unsafe sanitation                             0.1863171  0.19339884 -0.299672328 -0.016731663  0.19896063  0.018667674
No access to handwashing facility             0.1871333  0.21695026 -0.113298823 -0.003968245  0.14542773  0.072525541
Drug use - Sex: Both - Age: All Ages (Number) 0.1892845 -0.16541577  0.113301494 -0.334444259  0.49030011  0.059899391
Low bone mineral density                      0.1968279 -0.14878501 -0.087747638 -0.076008494  0.30120607 -0.057884162
Vitamin A deficiency                          0.1464584  0.32367765  0.401819235  0.023045455  0.18388913 -0.720363322
Child stunting                                0.1708111  0.28792107  0.122630355 -0.001275789 -0.12749176  0.139893522
Discontinued breastfeeding                    0.1749654  0.26687195  0.155543909 -0.030111741 -0.29185724  0.008988680
Non-exclusive breastfeeding                   0.1748693  0.27063690  0.105006508 -0.015724368 -0.20088979  0.294436078
Iron deficiency                               0.1854150  0.22570958 -0.114844898 -0.025326127 -0.01984031 -0.154687303
                                                      PC7          PC8         PC9        PC10         PC11         PC12
Outdoor air pollution                         -0.01580850  0.059986135 -0.32438377  0.14474716 -0.176834167  0.131566398
High systolic blood pressure                   0.02945960 -0.036930676  0.14881644 -0.05094277 -0.103525631 -0.082463390
Diet high in sodium                           -0.06416757 -0.076922216  0.14662610 -0.10353444  0.058544233 -0.122043002
Diet low in whole grains                       0.08967555 -0.121446761 -0.15305214 -0.16969820 -0.040602239  0.022612030
Alcohol use                                    0.13342777 -0.245967047  0.24982385 -0.06017592  0.521467891  0.145630371
Diet low in fruits                             0.04064596 -0.104034832  0.13800527 -0.17856516 -0.033237731 -0.401577109
Unsafe water source                            0.10977165 -0.135609818 -0.05256810  0.13474721 -0.170063938 -0.265693414
Secondhand smoke                              -0.06604279  0.105347078 -0.01782221  0.04194999 -0.187598253  0.052174955
Low birth weight                              -0.14811844  0.298219231 -0.23408396 -0.11678546  0.404458604 -0.012292543
Child wasting                                 -0.13408537 -0.087595507  0.20174980  0.03729790 -0.006729855  0.157811787
Unsafe sex                                     0.54192624  0.323601576 -0.08592691 -0.09809884  0.014081130 -0.089111222
Diet low in nuts and seeds                     0.39566109 -0.254703181 -0.21135460 -0.35939343 -0.011604923  0.004818479
Household air pollution from solid fuels      -0.07140012  0.158500071  0.27670719 -0.18947065  0.045267985  0.248701676
Diet low in vegetables                         0.01554542  0.317216575  0.54597106 -0.04707819 -0.163075969 -0.207184139
Low physical activity                          0.06407965  0.080285841 -0.11198047  0.48751972  0.008655809  0.003577027
Smoking                                       -0.11201573 -0.128790995 -0.01552891  0.02109561  0.206024543 -0.231979273
High fasting plasma glucose                   -0.07950075  0.145009370  0.15274083  0.20204816 -0.046031227  0.176120308
Air pollution                                 -0.04014487  0.096377332 -0.12295078  0.03411085 -0.108705295  0.177343210
High body-mass index                           0.02847792  0.163280162  0.05849540  0.06782682 -0.245747025  0.287883568
Unsafe sanitation                              0.15120563 -0.200073621 -0.10553531  0.11327452 -0.116656606 -0.031120371
No access to handwashing facility              0.10570018 -0.161542998  0.03971236  0.06225766 -0.009509537  0.276599387
Drug use - Sex: Both - Age: All Ages (Number) -0.45396212  0.002444566 -0.30904888 -0.40834551 -0.186464337  0.038189901
Low bone mineral density                      -0.05273384 -0.120669857 -0.11575597  0.44990084  0.288162303 -0.113230642
Vitamin A deficiency                           0.19479696 -0.001194183 -0.05599682  0.02602632 -0.189820023 -0.056330446
Child stunting                                -0.19416461 -0.345581207  0.06959963 -0.02369846  0.029048006  0.049050031
Discontinued breastfeeding                    -0.24756062  0.250969053 -0.12996267  0.03788576  0.070684151 -0.464926900
Non-exclusive breastfeeding                   -0.22328146 -0.285103663  0.09015743  0.06703746 -0.194617805  0.028191000
Iron deficiency                               -0.01690340  0.239191511 -0.10811798 -0.10881268  0.308347296  0.231170856
                                                     PC13        PC14         PC15         PC16        PC17         PC18
Outdoor air pollution                         -0.21120139 -0.05377484  0.079729670  0.116971899  0.07981823  0.179851457
High systolic blood pressure                   0.15639675  0.13937394 -0.104564479 -0.171277356 -0.22004049  0.156029164
Diet high in sodium                           -0.15888039  0.43435134 -0.033381656  0.058858275  0.19564390 -0.056323411
Diet low in whole grains                       0.23217378  0.04628123 -0.092850359 -0.278407481 -0.40296025  0.088882038
Alcohol use                                   -0.45370211 -0.08284986 -0.006047993  0.178771541 -0.33608518  0.064887695
Diet low in fruits                             0.21461770 -0.05885318 -0.067072388 -0.068957885 -0.01366792  0.082038099
Unsafe water source                           -0.10678632  0.20756714 -0.006113252 -0.173373356 -0.14249788  0.089115353
Secondhand smoke                              -0.09650363 -0.04532158  0.071298988  0.051415214 -0.09417417  0.014110707
Low birth weight                              -0.07160757  0.16083586  0.448319656 -0.254707377 -0.12995511  0.043031760
Child wasting                                  0.08336897  0.22430766 -0.096433687 -0.251182865  0.26113662 -0.430330618
Unsafe sex                                     0.13077885 -0.11545058  0.056647285 -0.048733380  0.03930808  0.011903459
Diet low in nuts and seeds                    -0.10008355 -0.22064695  0.001593869  0.111275178  0.43113780 -0.190938466
Household air pollution from solid fuels       0.28775004 -0.36717424 -0.019323798  0.008645935 -0.21629458 -0.352409359
Diet low in vegetables                        -0.08943735  0.02902952  0.195837075  0.280455345  0.14370606  0.152379789
Low physical activity                          0.36327944  0.13533064  0.186706116  0.447589571 -0.12064677 -0.180870756
Smoking                                        0.26538962  0.06535901  0.005549443 -0.036448562  0.18391500 -0.015778972
High fasting plasma glucose                   -0.05630049 -0.26887124 -0.007735577 -0.422882002  0.18560901  0.154534938
Air pollution                                 -0.03663755 -0.15991922  0.041108621  0.070455417 -0.02676403  0.009550123
High body-mass index                          -0.28774619  0.22718282 -0.102103224 -0.196572982  0.05043517 -0.101498919
Unsafe sanitation                             -0.08622762  0.19126824 -0.073817618 -0.004726821 -0.15601833 -0.207781146
No access to handwashing facility              0.04549806  0.12109005 -0.081350366  0.205560964 -0.10145888 -0.116534270
Drug use - Sex: Both - Age: All Ages (Number)  0.01335303  0.00674592  0.029302348  0.163524125 -0.02632112 -0.074822666
Low bone mineral density                       0.01203294 -0.26873076 -0.122302698 -0.169234166  0.21007042 -0.028366057
Vitamin A deficiency                          -0.09083109 -0.05929187  0.148998738 -0.036019534 -0.02664405  0.137315834
Child stunting                                 0.08223517 -0.04077055  0.530344535  0.012831625  0.17637699  0.124703347
Discontinued breastfeeding                    -0.30204146 -0.18905605 -0.298465750  0.113446046 -0.09365130 -0.291046734
Non-exclusive breastfeeding                    0.02038707 -0.25078333 -0.219273318  0.128876051 -0.05212550  0.348828850
Iron deficiency                                0.20728555  0.24119309 -0.442169527  0.180101289  0.23118694  0.413421388
                                                      PC19         PC20          PC21        PC22         PC23         PC24
Outdoor air pollution                          0.063376443  0.080870967 -0.2204373189 -0.11686245 -0.074708123 -0.390574392
High systolic blood pressure                  -0.038326828  0.023530741  0.0091608459 -0.12698129  0.230255793 -0.135131860
Diet high in sodium                           -0.036076767 -0.087739985  0.5490053642 -0.07959272 -0.122571853  0.052050906
Diet low in whole grains                      -0.122578865  0.270539777  0.1670926413  0.28288245 -0.264121323 -0.227056556
Alcohol use                                    0.093917478  0.025521351 -0.1704379166 -0.09974646  0.054578776 -0.080202077
Diet low in fruits                             0.149747645  0.011255932 -0.2196485992 -0.30546357  0.427121779  0.102662649
Unsafe water source                           -0.014285369  0.030984728  0.0106136057 -0.10904723 -0.087709811  0.045745556
Secondhand smoke                               0.047430636  0.177541263 -0.1064108269  0.53071934  0.366740786  0.446586888
Low birth weight                               0.307230767 -0.187282297  0.0815801728  0.05283483 -0.006113146  0.122474059
Child wasting                                  0.426668746  0.228100786 -0.1911547733  0.06210332  0.097801550 -0.299256678
Unsafe sex                                     0.002258092  0.047714154  0.0101374483 -0.01043616 -0.044756795  0.006076215
Diet low in nuts and seeds                     0.074369717 -0.078461342  0.1119232604  0.03648898  0.026254799  0.062362379
Household air pollution from solid fuels      -0.090872684 -0.003931715  0.0867345593 -0.19943052 -0.226599272  0.174186831
Diet low in vegetables                         0.034683255  0.151935959  0.0211801880  0.23082068 -0.220168851 -0.196344797
Low physical activity                          0.121594690 -0.092166431  0.1156994684 -0.20865933  0.086110528  0.031806010
Smoking                                       -0.143732166 -0.403190053 -0.5019077024  0.27226400 -0.359087688  0.031365931
High fasting plasma glucose                   -0.169496511 -0.448742478  0.2207878349 -0.02845321  0.252632576 -0.201891542
Air pollution                                  0.020804272  0.047511197 -0.1137708387 -0.13077163 -0.089969691 -0.215741365
High body-mass index                          -0.155514767  0.037787527 -0.2535861497 -0.22385734 -0.183090800  0.427278598
Unsafe sanitation                             -0.116010923 -0.155480753 -0.0128510217 -0.06736600 -0.090934829  0.040907612
No access to handwashing facility             -0.191073640 -0.240587833  0.0749554332  0.39526898  0.278441003 -0.138726950
Drug use - Sex: Both - Age: All Ages (Number)  0.067458382  0.012947341  0.0913024463 -0.04972685  0.064857790 -0.043880399
Low bone mineral density                       0.051434544  0.411768935  0.2155018796  0.09202182 -0.084063790  0.145011127
Vitamin A deficiency                           0.119609352 -0.061768124  0.0202268655  0.01406965 -0.026808043  0.040742049
Child stunting                                -0.481886121  0.261596448 -0.0386448958 -0.14704037  0.073905057  0.047838145
Discontinued breastfeeding                    -0.286788363 -0.011261198  0.0006888966  0.01126053  0.074694478 -0.109259975
Non-exclusive breastfeeding                    0.399766950 -0.202974039  0.1170030695  0.01223719 -0.246257195  0.204070714
Iron deficiency                               -0.136082827  0.136588625 -0.0449606269 -0.08005718  0.074990166  0.076022931
                                                       PC25         PC26          PC27          PC28
Outdoor air pollution                         -0.0960530080 -0.030543243 -0.0100195752 -0.5406502181
High systolic blood pressure                  -0.0983651954  0.491869559  0.6067595996 -0.0164008272
Diet high in sodium                           -0.0306606286 -0.078561637 -0.0761060883 -0.0003010180
Diet low in whole grains                      -0.0191710853 -0.162456409 -0.2875175504 -0.0062867313
Alcohol use                                    0.1733441858 -0.001626410 -0.0610814965  0.0067851223
Diet low in fruits                            -0.3108414755 -0.285380904 -0.3032475280 -0.0168783929
Unsafe water source                            0.5384088509 -0.340574455  0.2592353948  0.0197449037
Secondhand smoke                               0.2674283785  0.144957146 -0.1157757355 -0.0079057968
Low birth weight                              -0.1915154462  0.018102867  0.0727277467 -0.0062582240
Child wasting                                  0.1504853128  0.012563484 -0.0580580561 -0.0057254030
Unsafe sex                                     0.0183419574  0.020764802 -0.0127017915  0.0002144406
Diet low in nuts and seeds                     0.0963406073  0.064672197  0.1449146445  0.0084019297
Household air pollution from solid fuels       0.0985647284 -0.030417905  0.0892482804 -0.2583512562
Diet low in vegetables                        -0.0990851959  0.066834457 -0.0467261864  0.0066438097
Low physical activity                          0.1629254234 -0.045971825 -0.0562593893  0.0030369204
Smoking                                        0.0676398166  0.016242848  0.0769121528  0.0091671757
High fasting plasma glucose                    0.2147717764  0.005471424 -0.2171536935 -0.0095593383
Air pollution                                 -0.0628496153 -0.036332115  0.0271699756  0.7992281309
High body-mass index                          -0.2477046504 -0.178514582  0.0617186753  0.0036025249
Unsafe sanitation                             -0.1321356770  0.572560449 -0.4263170370  0.0105746775
No access to handwashing facility             -0.3738511265 -0.353127057  0.2276917250 -0.0228811869
Drug use - Sex: Both - Age: All Ages (Number)  0.0791655309  0.020442224  0.0331878632  0.0022192252
Low bone mineral density                      -0.2562109655  0.030655300  0.1409886171  0.0002457524
Vitamin A deficiency                          -0.0372235968 -0.019209027  0.0199366440 -0.0007243756
Child stunting                                 0.0226770853  0.036701474 -0.0368632076  0.0057222320
Discontinued breastfeeding                    -0.0002831245 -0.019569113  0.0353399183 -0.0011722978
Non-exclusive breastfeeding                   -0.0734871790  0.021714322 -0.0008594681 -0.0036823957
Iron deficiency                                0.1557323719  0.059839408 -0.0930480851  0.0046453223
```
we can see that we have the same results in terms of variance as the one we got in python. PC1 and PC2 explain the 96% of the variance of the dataset
```{r}
summary(pca_R) 

Importance of components:
                          PC1    PC2     PC3     PC4     PC5    PC6     PC7     PC8     PC9    PC10    PC11    PC12    PC13
Standard deviation     4.8308 1.9170 0.72642 0.49826 0.26292 0.2184 0.21497 0.14304 0.10384 0.08745 0.06630 0.04264 0.03844
Proportion of Variance 0.8334 0.1313 0.01885 0.00887 0.00247 0.0017 0.00165 0.00073 0.00039 0.00027 0.00016 0.00006 0.00005
Cumulative Proportion  0.8334 0.9647 0.98354 0.99241 0.99487 0.9966 0.99823 0.99896 0.99934 0.99962 0.99977 0.99984 0.99989
                          PC14    PC15    PC16    PC17    PC18    PC19    PC20     PC21    PC22     PC23     PC24     PC25
Standard deviation     0.02983 0.02405 0.02123 0.01562 0.01434 0.01238 0.01177 0.009678 0.00898 0.008367 0.007512 0.006411
Proportion of Variance 0.00003 0.00002 0.00002 0.00001 0.00001 0.00001 0.00000 0.000000 0.00000 0.000000 0.000000 0.000000
Cumulative Proportion  0.99992 0.99994 0.99996 0.99997 0.99998 0.99998 0.99999 0.999990 0.99999 1.000000 1.000000 1.000000
                           PC26    PC27      PC28
Standard deviation     0.003915 0.00362 0.0003472
Proportion of Variance 0.000000 0.00000 0.0000000
Cumulative Proportion  1.000000 1.00000 1.0000000
```
Now, lets do the PCA plot with R
```{r}
ggbiplot(pca_R)
```
![PCA R](https://github.com/user-attachments/assets/a58e9d32-9c47-48ff-bf84-874a4edbca82)

```{r}
ggbiplot(pca_R, 
         labels=deaths_risk_R$Entity) #this graphs shows whic Entity is every point
```
![PCA R 2](https://github.com/user-attachments/assets/c9595fba-64b6-41ce-b26a-7abc009174d7)

Thus, we can see in a similar way (but with more info since with R we can see which label has each point, which in our case is Entity) the same results and visualisations with R as with Python.

# Conclusions

Throughout this project we have seen how to perform Exploratory Data Analysis with both SQL and Python, the combination of which allows us to extract valuable insights through both queries and visualisations. After this we have seen how to reduce the dimensionality of a dataset. That is, we have seen an algorithm that allows us to focus on those variables that together define the largest proportion of variation in our data, allowing us to identify relationships between variables and focus on those that are really important.

Therefore, dimensionality reduction algorithms, in this case the PCA, allow us to easily focus, at least in a first stage, on those variables from which we can extract the most relevant information.

# References

[Number os Deaths by Risk Factor by Muhammad Umair A.B.](https://www.kaggle.com/datasets/muhammadumairab/number-of-deaths-by-risk-factor)

[PCA con Python by Joaquin Amat](https://cienciadedatos.net/documentos/py19-pca-python)

[PCA Analysis with R by Datacamp](https://www.datacamp.com/tutorial/pca-analysis-r)

[Principal Component Analysis in R Tutorial](https://www.datacamp.com/tutorial/pca-analysis-r)

[prcomp: Principal Components Analysis](https://www.rdocumentation.org/packages/stats/versions/3.6.2/topics/prcomp)

[Radviz plot](https://blog.finxter.com/radviz-in-pandas-plotting-how-it-works/)

[How to use SQL along with python](https://www.datacamp.com/es/tutorial/tutorial-how-to-execute-sql-queries-in-r-and-python)


