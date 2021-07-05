# The ODM Wide Table

The multiple tables of the data model make it easy to store various types of data. Though the ODM structure allows for flexibility and specificity when committing data, this structure is not optimal for consulting and displaying the data, especially when trying to follow the evolution of different quantities over time. To serve this purpose, we propose a systematic template to generate a table that distributes the various recorded time series from a given site into separate columns. The data is spread across rows according to time. We call this table, the "wide" table, since it spreads data horizontally into individual columns.

## Purpose

The wide table is meant to conveniently lay out the information collected at a specific **site**. 

The main action taken at any site is the collection of samples. These samples, in turn, undergo different wastewater measurements. In parallel to this sampling campaigns, other measurements me be happening at the same site (site measurements) that can contextualize the WBE campaign. And in parallel to any measurement occurring at the sampling site, public health authorities are also collecting case data - not at the sampling site itself, but rather within a health region or a sewershed.

Thus, to create a wide table, time series data from those three sources must be aligned and joined.

The `Wide.csv` file describes the creation procedure of a wide table. It does not, however, list the actual column names of the wide table, as those will be different based on the types of data that are gathered, their units, their aggregation method, etc. Instead, the file provides a system to automatically determine a unique name for each type of data present in the ODM.

## How to read the `Wide.csv` file

1. `tableSubject`

    WBE involves the cross-referencing data from two different spatial regions: the health region where COVID-19 cases are being counted, and the sewershed region that is drained by the sampling site. Since both of these regions of interest are recorded in the `Polygon` table, the `tableSubject` assigns a prefix to the column names coming out of the `Polygon` table to distinguish info related to the health data (`CPHD`) or to wastewater data (`WW`).

1. `tableName`

    The name of the ODM table where the data is originally stored. 

1. `tableNickname`

    A short version of the ODM table name. The `tableNickname` is used to name wide table columns to save character space in the column names.

1. `descColumns`

    Description columns hold metadata relevant to the measures present in the wide table.

1. `valueColumns`

    Value columns hold data that can be expressed as a time series. 

1. `valueColumnFormula`

    The name of the value columns from the original 'long' tables can't be placed into the wide table directly because they are too generic. They need to be supplemented using information stored inside other columns of the original ODM table. `valueColumnFormula` shows the "recipe" that should be followed to create a column name based on the values stored in different columns of the long table. The names of the different elements are denoted by a `$` symbol. Elements of the column name are separated by an underscore `_`. 

    In some situations, elements of the name that belong in a certain position inside the name can come from one of several columns or be absent. In those cases, a parenthesis is placed in the formula, with the different options separated by the keyword `OR`. If a position in the column name can be removed, the keyword `NULL` is presented as an option in the `OR` block.

1. `descColumnFormula`

    This column defines how description columns should be renamed when placed
inside a wide table.


1. `timestampFormula`

    The data placed inside the wide table is aligned by time. Since time information is stored differently across ODM tables, the column `timestampFormula` is included in the wide table description to prescribe how to calculate the timestamp that should drive the alignment of public health data, site measurements or WBE sample collection and their measurements. For certain tables, different types of data have different formulae. In that case, an `IF` block is used to separate the appropriate formula for each type. These statements take the form:
    
     ```IF(condition, formula if true, formula if false)```.

1. `joinFormula`

    Given that the wide table merges data from several tables, the `joinFormula` describes how these tables should be joined for them to be aligned properly. A `joinFormula` takes the form:

    ```typeOfJoin(leftTableNickname_ColumnToJoinOn -> rightTableNickname_ColumnToJoinOn WHERE tableNickname_column=value)```

## Steps for creating a wide table

1. Pick a site and find out its `siteID` based on the `Site` table. 

1. Pick a health region with cases data to match against your WBE samples. This choice must be made manually because public health data and wastewater surveillance data do not typically survey the exact same geographical region. The creator of a wide table therefore has to decide on how they wish to match these related but independent data sources. Once a choice of public health region has been made, find its `polygonID` in the `Polygon` table.

1. Create a wide version of all the tables that will be used to create the final wide table. The create these, transform the tables by creating each column prescribed by the `valueColumnFormula`, `descColumnFormula` and `timestampFormula` for that table.

1. Filter the `Sample` table by `siteID` so that only samples collected at the site of interest remain.

1. Join the widened and filtered `Sample` table with the widened `WwMeasure` table.

1. Join the resulting table with the widened `Site` table.

1. Join the resulting table with the widened `SiteMeasureTable`.

1. Join the resulting table with a copy of the widened `Polygon` table based on the studied sewershed.

1. Join the resulting table with another copy of the widened `Polygon` table based on the studied health region.

1. Join the resulting table with the `CovidPublicHealthData` table based on the studied health region.

