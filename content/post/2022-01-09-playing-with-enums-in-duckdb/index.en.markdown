---
title: Playing with Enums in DuckDB
author: edalfon
date: '2022-01-09'
slug: playing-with-enums-in-duckdb
categories:
  - R
tags:
  - DuckDB
description: Do use ENUM to get your data into DuckDB
image: /p/playing-with-enums-in-duckdb/index.en_files/figure-html/enum-res-plot-1.svg
math: ~
license: ~
hidden: no
comments: yes
---



Now, let's try DuckDB's ENUM type. Since they announced this feature, I wanted to try it out, because most of the datasets I have to deal with have a bunch of string columns, frequently, low-cardinality categorical variables. So I would expect substantial gains in query performance and database size, by using enums. For reference, here links to the [blog post](https://duckdb.org/2021/11/26/duck-enum.html) and [documentation](https://duckdb.org/docs/sql/data_types/enum) describing enums.









Once again, DuckDB follows PostgreSQL's syntax.




```sql
CREATE TYPE sex_enum AS ENUM ('F', 'M');
```

After creating the enum type, you can use it as other built-in types, 
for example, to cast a column into the new type.


```sql
SELECT sex::sex_enum FROM tiny_data LIMIT 10;
```


<div class="knitsql-table">


Table: Table 1: Displaying records 1 - 10

|CAST(sex AS sex_enum) |
|:---------------------|
|F                     |
|F                     |
|F                     |
|F                     |
|F                     |
|F                     |
|M                     |
|F                     |
|M                     |
|F                     |

</div>

But it seems you cannot simply change a column's type and make it an enum, using an ALTER TABLE statement. None of the statements below work for enums as the target data type, but they would work for varchar as the target data type.


```sql
ALTER TABLE tiny_data ALTER sex TYPE sex_enum;
```


```sql
ALTER TABLE tiny_data ALTER sex SET DATA TYPE sex_enum USING sex;
```


```sql
ALTER TABLE tiny_data ALTER sex 
SET DATA TYPE sex_enum USING CAST(sex AS sex_enum);
```

It makes sense that simply changing the data type does not work. After all, 
enums have a different underlying representation. But there is a workaround: create a new table and update its values to those of the original varchar column.


```sql
ALTER TABLE tiny_data ADD COLUMN sex_2 sex_enum;
```


```sql
UPDATE tiny_data SET sex_2 = sex::sex_enum;
```


```sql
SELECT sex, sex_2 FROM tiny_data LIMIT 10;
```


<div class="knitsql-table">


Table: Table 2: Displaying records 1 - 10

|sex |sex_2 |
|:---|:-----|
|F   |F     |
|F   |F     |
|F   |F     |
|F   |F     |
|F   |F     |
|F   |F     |
|M   |M     |
|F   |F     |
|M   |M     |
|F   |F     |

</div>

That works!, but it's not ideal. So I would prefer to directly ingest the data
into an enum column. Let's see how to do that.


```sql
CREATE TABLE tiny_data_sex AS
SELECT sex::sex_enum --cast into the enum type
FROM read_csv_auto('D:/tiny_data.txt') --read from the csv file
```


```sql
SELECT * FROM tiny_data_sex LIMIT 10;
```


<div class="knitsql-table">


Table: Table 3: Displaying records 1 - 10

|CAST(sex AS sex_enum) |
|:---------------------|
|F                     |
|F                     |
|F                     |
|F                     |
|F                     |
|F                     |
|M                     |
|F                     |
|M                     |
|F                     |

</div>

Good! Creating the table with the enum data type and then using a COPY statement also works.

There are a couple of slight inconveniences, though. To take advantage of enums, you need to know all unique values in advance and explicitly create the enum type before reading the data from the csv. Both are minor issues, but they can slow you down. 
- The need to explicitly create the enum type it's just one more step before you can play with the data. And quite often -when there are more than a handful of unique values- it would involve reading the values from some other place and deal with the potential issues in that process (missing or invalid values and so on).
- The need to know all unique values in advance could be a bit more troublesome. First, even though ideally you should always have a reference table for such columns, very often you just get the CSV file and no reference tables at all for the categorical variables. And sometimes even when you think you know the unique values, things can go south if there are errors in the data, or in your assumption for unique values. And example: the sex enum was created with values Female and Male, but it turns out in the last year, now they allow a new category; non-binary. Then it fails ingesting data for that year. And you would need to also ingest again the data from the previous years, because enums are currently static and you cannot update the enum to make it compatible with the data for the last year.

Luckily, the DuckDB team has in the roadmap improvements for both issues above. They have in mind to automatically detect those kind of columns and handle them as enums. And they also plan to allow insertion and removal of values into the enum. That sounds great. Hopefully it has some priority and we can benefit from such improvements soon.

# Results

Ok, let's see the benefits of ENUM in my playground dataset (Figure below). Basically, it helps cut the database size in 30-40% and also help with the ingest time for large data.

<img src="{{< blogdown/postref >}}index.en_files/figure-html/enum-res-plot-1.svg" width="85%" style="display: block; margin: auto;" />

# Wrap-up

- ENUM seem awesome, and it has a promising roadmap in DuckDB. 
- Please, do use them, even though there might be a couple of slight inconveniences, the benefits in terms of database size, ingest and query times clearly outweigh those costs



