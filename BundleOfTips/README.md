# Summary

This article details a set of various tips & tricks in Power Query (M language) and DAX.

## Multi-criteria filter

When multiple criteria must be evaluated for each row, the keyword `and`, `or` or `not`.

``` powerquery
    ...
    FilteredRows = Table.SelectRows(PreviousStep, each ([column1] <> "criteria1" and [column2] <> null or [column3]) <> "criteria1"),
    ...
```

[Power Query (M language) operators](https://docs.microsoft.com/en-us/powerquery-m/operators)

## Remove duplicates in Power Query

A proper *remove duplicates* implementation in Power Query requires 4 steps:
- First: make sure that you have removed any columns from the dataset that might make a row unique (e.g. an identifier, a date) that should be considered a duplicate,
- Second: sort the data  according to required criteria,
- Third: **buffer the entire table**. Otherwise, the third step could lead to undesirable and inaccurate results,
- Fourth: remove the duplicates.

``` powerquery
    ...
    RemovedColums = Table.RemoveColumns(PreviousStep,{"Column1", "Column2"}),
    SortedRow = Table.Sort(RemovedColums,{{"Criterion1", Order.Ascending}, {"Criterion2", Order.Descending}}),
    BufferedTable = Table.Buffer(SortedRow),
    RemovedDuplicates = Table.Distinct(BufferedTable),
    ...
```

## Agregation in Power Query

A proper *agregation* implementation in Power Query, like *remove duplicates* implementation, requires 4 steps:
- First: make sure that you have removed any columns from the dataset that might make a row unique (e.g. an identifier, a date) that should be considered a duplicate,
- Second: sort the data  according to required criteria,
- Third: **buffer the entire table**. Otherwise, the third step could lead to undesirable and inaccurate results,
- Fourth: group the rows and calculate.

``` powerquery
    ...
    RemovedColums = Table.RemoveColumns(PreviousStep,{"Column1", "Column2"}),
    SortedRow = Table.Sort(RemovedColums,{{"Criterion1", Order.Ascending}, {"Criterion2", Order.Descending}}),
    BufferedTable = Table.Buffer(SortedRow),
    GroupedRows = Table.Group(BufferedTable, {"GroupByCriterionField1", "GroupByCriterionField2"},
        {
            {"GroupedField1", each List.Sum([FieldToSum]), type nullable number},
            {"GroupedField2", each List.Min([MinField]), type nullable number},
            {"GroupedField3", each List.Count([FieldToCount]), type nullable number}
        }),
    ...
```

> [*List* functions](https://docs.microsoft.com/en-us/powerquery-m/list-functions)

## Join Tables

Joining two tables in Power Query is done in two steps:
1. A nested join operation (`Table.NestedJoin`) is added. It takes 6 parameters:
    - The source table,
    - The list of fields of the source table used to join the two tables,
    - The target table,
    - The matching list of fields of the target table used to join the two tables,
    - The alias used for the fields from the target table,
    - The type of join (`JoinKind`):
        * [Inner](https://docs.microsoft.com/en-us/powerquery-m/joinkind-inner)
        * [LeftAnti](https://docs.microsoft.com/en-us/powerquery-m/joinkind-leftanti)
        * [LeftOuter](https://docs.microsoft.com/en-us/powerquery-m/joinkind-leftouter)
        * [RightAnti](https://docs.microsoft.com/en-us/powerquery-m/joinkind-rightanti)
        * [RightOuter](https://docs.microsoft.com/en-us/powerquery-m/joinkind-rightouter)
        * [FullOuter](https://docs.microsoft.com/en-us/powerquery-m/joinkind-fullouter)

2. An expansion operation (`ExpandTableColumn`) is then added, to get the desired fields in the dataset. The operation takes 4 parameters:
    - The source table (the *nested join* operation result),
    - The target alias specified in the *nested join* operation,
    - The list of fields from the target table to add to the dataset,
    - The list of final name of the fields added to the dataset

``` powerquery
    ...
    MergedQueries = Table.NestedJoin(PreviousStep, {"Column1InSource", "Column2InSource"}, TargetTable, {"Column1InTarget", "Column2InTarget"}, "TargetAlias", JoinKind.Inner),
    ExpandFields = Table.ExpandTableColumn(MergedQueries, "TargetAlias", {"Column1InTarget", "Column2InTarget"}, {"Column1NameInFinalDataset", "Column2NameInFinalDataset"}),
    ...
```

## Refresh date

It can be convenient to have the last refresh data date (time).

Here is how to create a simple table:

``` powerquery
let
    RefreshDate = 
    let
        Source = Date.From(DateTime.LocalNow()),
        ConvertedtoTable = #table(1, {{Source}}),
        RenamedColumns = Table.RenameColumns(ConvertedtoTable,{{"Column1", "RefreshDate"}}),
        ChangedType = Table.TransformColumnTypes(RenamedColumns,{{"RefreshDate", type date}})
    in
        ChangedType
in
    RefreshDate
```

With the date and time:
``` powerquery
let
    RefreshDate = 
    let
        Source = Date.From(DateTime.LocalNow()),
        ConvertedtoTable = #table(1, {{Source}}),
        RenamedColumns = Table.RenameColumns(ConvertedtoTable,{{"Column1", "RefreshDateTime"}}),
        ChangedType = Table.TransformColumnTypes(RenamedColumns,{{"RefreshDateTime", type date}})
    in
        ChangedType
in
    RefreshDate
```

Then, the following measure `LastRefreshDate` can be defined:

 ``` DAX
MAX(RefreshDate[RefreshDate])
```

or

 ``` DAX
MAX(RefreshDate[RefreshDateTime])
```
