---
title: Test-driving DuckDB and Arrow in a poorman's laptop
author: edalfon
date: '2021-12-05'
categories:
  - R
tags:
  - Arrow
  - DuckDB
  - PostgreSQL
slug: test-driving-duckdb-and-arrow
image: https://i.imgur.com/OEagu52.png
hidden: no
comments: yes
---

<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index.en_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/crosstalk/js/crosstalk.min.js"></script>

Whenever I have to analyze largish[^1] data in my laptop, I instinctively
fire up my good old PostgreSQL server running locally, ingest the data in there
and breezily wrangle my way through the data using SQL. Postgres may not be the
fastest alternative out there. But it is feature-rich: you can throw anything
at it, and it just works. It’s very hard to make it choke with an out-of-memory
error and, by default, runs just fine in resource-constrained environments
(e.g. in a Raspberry Pi).

I’ve kept an eye on [DuckDB](https://duckdb.org/) and [Apache
Arrow](https://arrow.apache.org/) as other open-source alternatives for the
workflow outlined above. Both share a number of features that make them very
appealing for analytical workloads in general, and for the in-laptop-analytics
in particular:[^2]

-   Columnar format and vectorized operations
-   In-process libraries (no need to install a server)
-   Handle larger-than-memory datasets
-   APIs available for several languages, including R and Python
-   Open-source

I did try both DuckDB and Arrow some time ago (very quickly, though). But I do
not use them routinely. At least not just yet. When I first tried them, Arrow
did not support grouped aggregations and joins and that was a no-go for me.
DuckDB did not have support for grouping sets or filtered aggregation (or at
least I could not get it to work) and I found it slow ingesting data from CSV
files (much slower than Postgres in that, even though queries were indeed
faster).

But both projects move at a very fast pace and now I see release news where
[Arrow 6.0.0 “adds grouped aggregation and joins in the dplyr interface, on top
of the new Arrow C++ query
engine”](https://arrow.apache.org/blog/2021/11/04/6.0.0-release/) and [DuckDB
announced “support for GROUPING SETS, ROLLUP,
CUBE”](https://github.com/duckdb/duckdb/releases/tag/v0.3.1). So it is time to
take both projects for a spin again (and try and document the test-drive here).

# The test-drive

Ultimately, I would like to run an existing data wrangling pipeline using
DuckDB and Arrow as backends, and compare running time using PostgreSQL as
backend. For this, DuckDB is particularly appealing because they use PostgreSQL
parser and support a similar feature set. So, ideally, the pipeline can run on
DuckDB simply by changing the connection without having to change the queries.
That did not really work as smoothly, so I will be trying step-by-step: i)
ingest the data, ii) try some aggregate queries, iii) joins, …, and let’s see
how it goes.

## The data

For this test it was key to use real data. Meaning a large data set that I
would actually have to use / have used for analysis in my laptop. Of course I
could have simply used the popular NYC Taxi trip record data or any other large
data set publicly available. But that would have been missing the point. I
wanted to keep this exercise as close as possible to an actual use case for me.
After all, this is just an informal test to see how to integrate DuckDB and/or
Arrow in my workflow and to assess the potential benefits, for example, in
terms of processing time.[^3]

I will use a few samples of a larger data set[^4] that contains 25 columns (2 numeric, 2 timestamp, 10 integer and the rest text, most of them categorical variables, some with low cardinality -e.g. sex-, others with millions of different values). I will use one round of data (\~500M rows) stored
in a csv file (uncompressed 82GB). First test in smaller samples (tiny, small, mid) before trying out using one whole round of data.

-   Tiny data: 1M rows, csv uncompressed size \~170MB
-   Small data: 10M rows, csv uncompressed size \~1,7GB
-   Mid data: 70M rows, csv uncompressed size \~10GB
-   Round data: \~500M rows, csv uncompressed size \~82GB

## The code and software

I will be running this test from R, using `{targets}` to orchestrate the
different steps and `{bench}` to benchmark code snippets and measure execution
time.

-   DuckDB version 0.3.0 \[I did not pay attention and ran this using 0.3.0 and then I noticed 0.3.1 is already available, which enables by default multi-threading; so perhaps I should re-run this with 0.3.1 to see if it changes the results\]
-   Arrow version 6.0.0
-   PostgreSQL version 14.1

Initially I will be using DuckDB and Arrow without fiddling with any configuration (no custom memory_limit or threads; just the defaults). In PostgreSQL, however, I typically tweak a couple of configuration parameters, in this case, `shared_buffers=2GB` and `work_mem=1GB` (see this [link](https://www.enterprisedb.com/postgres-tutorials/how-tune-postgresql-memory) for some useful rules-of-thumb).

The code for the whole exercise is available in this [Github repo](https://github.com/edalfon/backend_compare)

## A poorman’s laptop

Running this may take a while, so I cannot really use my daily driver laptop. But I do have an old laptop lying around that can be used for this.
It’s a 10-years old Dell XPS 17. I guess at the time it was a good machine:
Processor Intel i7-2630QM, 8GB DDR3 RAM, SSD SATA, nowadays running Windows 10
Home. But for today’s standards, it can be considered a resource-constrained
environment.

So it’s important to keep in mind the kind of machine running this exercise. Not sure about this, but I would consider the results as a lower bound to
the performance improvements that DuckDB and Arrow could bring (e.g. Arrow
explicitly try to take advantage of “modern hardware”). It will be interesting
to see how much of a difference modern hardware would make. Hopefully soon, I
will get my hands on a new fancy MacBook Pro to replicate this test.

# Ingesting data

## What will be tested

I will compare the time it takes to ingest the raw data from the csv files into
each of the backends.

### Arrow

In the case of Arrow, let’s try with the two key file formats: i. parquet and ii. arrow (aka ips/feather; see `?arrow:open_dataset`). It boils down to use a couple of functions from the `{arrow}` R package: `open_dataset()` and `write_dataset()`, like this:

``` r
  csv_ds <- arrow::open_dataset(
    sources = src_file,
    format = "text",
    delimiter = ";"
  )
  
  arrow::write_dataset(
    dataset = csv_ds,
    path = arrow_path,
    format = "arrow" # and "parquet"
  )
```

### DuckDB

For DuckDB, probably the simplest way is to use a `CREATE TABLE AS` statement with a query reading directly from the csv file using `read_csv_auto()`. This lets DuckDB detect everything (column names, delimiter, data types, etc.).

``` sql
CREATE TABLE table_name AS
SELECT * 
FROM read_csv_auto('src_file.csv')
;
```

An alternative is the `COPY` statement. You would have to type more to “manually” create the table and decide on the data type for each column doing something like this: `CREATE TABLE table_name ("col1" TEXT, ...);`. And after that just copy the csv into that table:

``` sql
COPY table_name
FROM 'src_file.csv' ( DELIMITER ';', HEADER )
;
```

A third alternative [described in the documentation](https://duckdb.org/docs/data/csv) (hereafter labelled as copy2) is a slight variation of the COPY statement to let DuckDB detect the format “and omit the otherwise required configuration options.”

``` sql
COPY table_name
FROM 'src_file.csv' ( AUTO_DETECT TRUE )
;
```

Let’s compare these three approaches, just to take a look if there is any overhead associated to auto detection of csv format and data types.

### PostgreSQL

For Postgres, let’s also compare the COPY statement approach, which uses very similar syntax as in DuckDB

``` sql
COPY table_name
FROM 'src_file.csv' (FORMAT 'csv', DELIMITER ';', HEADER )
;
```

And an alternative using a foreign table and csv foreign data wrapper.

``` sql
CREATE EXTENSION IF NOT EXISTS file_fdw; 
CREATE SERVER IF NOT EXISTS csv_src FOREIGN DATA WRAPPER file_fdw;
CREATE FOREIGN TABLE table_name_fg (
  col1 STRING,
  ...
)
SERVER csv_src
OPTIONS (
  FILENAME 'src_file.csv',
  FORMAT 'csv',
  HEADER 'TRUE',
  DELIMITER ';',
  ENCODING 'UTF-8'
)
;

CREATE TABLE table_name AS SELECT * FROM table_name_fg;
```

## Results: How long it takes to ingest such data?

Well, for this exercise Arrow is fast!, way faster than Postgres, but also faster than DuckDB. Arrow native file format is also faster than parquet. Both the tests using tiny and small data show very similar results, where arrow file format is the fastest and compared to that it takes around:

-   1.6x longer to ingest into parquet
-   5x longer to ingest into DuckDB
-   15x longer to ingest into PostgreSQL

There are no major differences associated to the approach used to ingest in each backend (so, no major overhead for auto-detecting file format and data types in DuckDB).

<img src="https://i.imgur.com/QKE8ezR.png" width="50%" /><img src="https://i.imgur.com/OEagu52.png" width="50%" />

<details>
<summary>
{bench} details for tiny data
</summary>
<table class=" lightable-paper" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
expression
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
median
</th>
<th style="text-align:right;">
itr/sec
</th>
<th style="text-align:right;">
mem_alloc
</th>
<th style="text-align:right;">
gc/sec
</th>
<th style="text-align:right;">
n_itr
</th>
<th style="text-align:right;">
n_gc
</th>
<th style="text-align:right;">
total_time
</th>
<th style="text-align:left;">
result
</th>
<th style="text-align:left;">
memory
</th>
<th style="text-align:left;">
time
</th>
<th style="text-align:left;">
gc
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
arrow
</td>
<td style="text-align:right;">
1.3s
</td>
<td style="text-align:right;">
1.31s
</td>
<td style="text-align:right;">
0.7604814
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.4345608
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
9.21s
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
1.836621, 1.467352, 1.328626, 1.314380, 1.420574, 1.326711, 1.312669, 1.297491, 1.298764, 1.326055
</td>
<td style="text-align:left;">
2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
parquet
</td>
<td style="text-align:right;">
2.09s
</td>
<td style="text-align:right;">
2.13s
</td>
<td style="text-align:right;">
0.4681122
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
21.36s
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
2.272508, 2.125310, 2.138580, 2.101764, 2.098363, 2.136137, 2.157207, 2.126433, 2.114051, 2.092045
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb
</td>
<td style="text-align:right;">
5.15s
</td>
<td style="text-align:right;">
6.79s
</td>
<td style="text-align:right;">
0.1488356
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.12m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
5.153599, 6.783404, 6.725932, 6.822941, 6.752167, 6.830131, 6.721660, 6.902978, 6.793911, 7.701503
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy
</td>
<td style="text-align:right;">
8.33s
</td>
<td style="text-align:right;">
8.42s
</td>
<td style="text-align:right;">
0.1179548
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.41m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
8.416875, 8.519529, 8.361855, 8.623358, 8.420646, 8.392340, 8.334223, 8.698937, 8.412829, 8.597660
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy2
</td>
<td style="text-align:right;">
6.47s
</td>
<td style="text-align:right;">
7.15s
</td>
<td style="text-align:right;">
0.1311416
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0145713
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1.14m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
8.828867, 8.365383, 8.991698, 8.479016, 6.709015, 6.474999, 6.769448, 7.149739, 6.826154, 6.742778
</td>
<td style="text-align:left;">
0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_foreign
</td>
<td style="text-align:right;">
19.36s
</td>
<td style="text-align:right;">
25.09s
</td>
<td style="text-align:right;">
0.0431928
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0047992
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
3.47m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
21.76691, 19.67458, 19.53117, 19.52908, 19.36306, 25.09045, 26.06149, 26.48230, 26.44349, 26.19228
</td>
<td style="text-align:left;">
1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_copy
</td>
<td style="text-align:right;">
22.02s
</td>
<td style="text-align:right;">
22.07s
</td>
<td style="text-align:right;">
0.0452669
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0050297
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
3.31m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
22.01945, 22.23197, 22.09389, 22.03953, 22.18977, 22.06471, 22.11497, 22.01474, 22.03280, 22.13380
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
</tbody>
</table>
</details>
<details>
<summary>
{bench} details for small data
</summary>
<table class=" lightable-paper" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
expression
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
median
</th>
<th style="text-align:right;">
itr/sec
</th>
<th style="text-align:right;">
mem_alloc
</th>
<th style="text-align:right;">
gc/sec
</th>
<th style="text-align:right;">
n_itr
</th>
<th style="text-align:right;">
n_gc
</th>
<th style="text-align:right;">
total_time
</th>
<th style="text-align:left;">
result
</th>
<th style="text-align:left;">
memory
</th>
<th style="text-align:left;">
time
</th>
<th style="text-align:left;">
gc
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
arrow
</td>
<td style="text-align:right;">
12.5s
</td>
<td style="text-align:right;">
16.27s
</td>
<td style="text-align:right;">
0.0658646
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
2.53m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
14.26708, 12.57470, 12.76333, 12.50327, 16.12645, 16.42233, 16.75937, 16.68181, 16.63274, 17.09561
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
parquet
</td>
<td style="text-align:right;">
26.76s
</td>
<td style="text-align:right;">
27.37s
</td>
<td style="text-align:right;">
0.0362535
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
4.6m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
28.99158, 27.96898, 26.95793, 26.76077, 27.31003, 27.87820, 27.42645, 26.93932, 28.47725, 27.12517
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb
</td>
<td style="text-align:right;">
1.15m
</td>
<td style="text-align:right;">
1.42m
</td>
<td style="text-align:right;">
0.0120938
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0013438
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
12.4m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
96.78378, 85.15403, 85.53791, 85.18231, 68.82363, 73.90329, 85.23597, 86.54822, 87.04882, 86.74796
</td>
<td style="text-align:left;">
1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy
</td>
<td style="text-align:right;">
1.19m
</td>
<td style="text-align:right;">
1.23m
</td>
<td style="text-align:right;">
0.0129047
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
12.91m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
86.58523, 87.16100, 71.94344, 71.47411, 72.32618, 72.20082, 73.62987, 73.58680, 75.32237, 90.67873
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy2
</td>
<td style="text-align:right;">
1.26m
</td>
<td style="text-align:right;">
1.51m
</td>
<td style="text-align:right;">
0.0116235
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
14.34m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
90.50274, 91.91102, 91.19395, 91.24822, 91.36625, 92.39341, 84.57364, 75.66488, 75.84726, 75.62813
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_foreign
</td>
<td style="text-align:right;">
2.98m
</td>
<td style="text-align:right;">
4m
</td>
<td style="text-align:right;">
0.0045031
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0005003
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
33.31m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
233.7513, 240.4563, 239.7256, 196.6095, 202.6231, 241.5512, 241.1700, 186.1951, 178.4919, 240.6790
</td>
<td style="text-align:left;">
0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_copy
</td>
<td style="text-align:right;">
2.72m
</td>
<td style="text-align:right;">
3.55m
</td>
<td style="text-align:right;">
0.0049937
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
33.38m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
219.6726, 215.4499, 163.2316, 184.3710, 221.4437, 219.7841, 185.1851, 163.6718, 210.6791, 219.0370
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
</tbody>
</table>
</details>

Interestingly, when the data are somewhat larger, at some point DuckDB’s performance degrades and falls behind Postgres. While the other relative times
remain in the same ballpark (1.6x parquet, \~12x Postgres), now it takes almost 25x longer for DuckDB to ingest the data, compared to arrow. In absolute numbers, while Arrow takes less than 2 minutes to ingest the data, DuckDB takes about 40 minutes (and Postgres less than 20 minutes).

<img src="https://i.imgur.com/eBWiY3z.png" width="480" style="display: block; margin: auto;" />

<table class=" lightable-paper" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
expression
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
median
</th>
<th style="text-align:right;">
total_time
</th>
<th style="text-align:right;">
n_itr
</th>
<th style="text-align:right;">
n_gc
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
arrow
</td>
<td style="text-align:right;">
1.58m
</td>
<td style="text-align:right;">
1.61m
</td>
<td style="text-align:right;">
13.02m
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
4
</td>
</tr>
<tr>
<td style="text-align:left;">
parquet
</td>
<td style="text-align:right;">
2.85m
</td>
<td style="text-align:right;">
2.98m
</td>
<td style="text-align:right;">
30.54m
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb
</td>
<td style="text-align:right;">
37.6m
</td>
<td style="text-align:right;">
39.75m
</td>
<td style="text-align:right;">
6.66h
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy
</td>
<td style="text-align:right;">
37.43m
</td>
<td style="text-align:right;">
39.21m
</td>
<td style="text-align:right;">
6.5h
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy2
</td>
<td style="text-align:right;">
38.52m
</td>
<td style="text-align:right;">
39.34m
</td>
<td style="text-align:right;">
5.93h
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_foreign
</td>
<td style="text-align:right;">
18.77m
</td>
<td style="text-align:right;">
19.26m
</td>
<td style="text-align:right;">
2.91h
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_copy
</td>
<td style="text-align:right;">
17.64m
</td>
<td style="text-align:right;">
17.89m
</td>
<td style="text-align:right;">
2.75h
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
</tr>
</tbody>
</table>
<details>
<summary>
{bench} details for mid data
</summary>
<table class=" lightable-paper" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
expression
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
median
</th>
<th style="text-align:right;">
itr/sec
</th>
<th style="text-align:right;">
mem_alloc
</th>
<th style="text-align:right;">
gc/sec
</th>
<th style="text-align:right;">
n_itr
</th>
<th style="text-align:right;">
n_gc
</th>
<th style="text-align:right;">
total_time
</th>
<th style="text-align:left;">
result
</th>
<th style="text-align:left;">
memory
</th>
<th style="text-align:left;">
time
</th>
<th style="text-align:left;">
gc
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
arrow
</td>
<td style="text-align:right;">
1.58m
</td>
<td style="text-align:right;">
1.61m
</td>
<td style="text-align:right;">
0.0102385
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0051192
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
13.02m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
115.88758, 103.35154, 100.03381, 97.96673, 101.98887, 97.18950, 95.27126, 94.86438, 95.85733, 94.87548
</td>
<td style="text-align:left;">
2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
parquet
</td>
<td style="text-align:right;">
2.85m
</td>
<td style="text-align:right;">
2.98m
</td>
<td style="text-align:right;">
0.0054566
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
30.54m
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
175.7699, 183.1974, 209.0024, 176.9291, 171.2705, 176.0256, 171.7868, 191.0403, 181.2983, 196.3368
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb
</td>
<td style="text-align:right;">
37.6m
</td>
<td style="text-align:right;">
39.75m
</td>
<td style="text-align:right;">
0.0004170
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
6.66h
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
2497.150, 2255.792, 2297.318, 2384.956, 2308.614, 2379.006, 2589.718, 2469.575, 2384.735, 2415.811
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy
</td>
<td style="text-align:right;">
37.43m
</td>
<td style="text-align:right;">
39.21m
</td>
<td style="text-align:right;">
0.0004274
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
6.5h
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
2354.093, 2389.514, 2415.575, 2351.088, 2245.535, 2364.025, 2281.866, 2282.393, 2388.673, 2321.990
</td>
<td style="text-align:left;">
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
duckdb_copy2
</td>
<td style="text-align:right;">
38.52m
</td>
<td style="text-align:right;">
39.34m
</td>
<td style="text-align:right;">
0.0004216
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000468
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
5.93h
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
2311.412, 2445.555, 2360.389, 2359.646, 2393.704, 2371.317, 2354.011, 2467.153, 2351.584, 2379.501
</td>
<td style="text-align:left;">
0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_foreign
</td>
<td style="text-align:right;">
18.77m
</td>
<td style="text-align:right;">
19.26m
</td>
<td style="text-align:right;">
0.0008593
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0000955
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2.91h
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
1249.875, 1142.859, 1145.987, 1125.913, 1171.138, 1182.262, 1204.312, 1131.437, 1214.214, 1155.513
</td>
<td style="text-align:left;">
1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
<tr>
<td style="text-align:left;">
pg_copy
</td>
<td style="text-align:right;">
17.64m
</td>
<td style="text-align:right;">
17.89m
</td>
<td style="text-align:right;">
0.0009091
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.0001010
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2.75h
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
NULL
</td>
<td style="text-align:left;">
1062.921, 1107.936, 1058.463, 1080.978, 1109.508, 1061.997, 1071.106, 1073.457, 1138.315, 1216.212
</td>
<td style="text-align:left;">
0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
</td>
</tr>
</tbody>
</table>
</details>

The results ingesting a whole round of data[^5] (\~500 million rows in a single csv file, 82GB) are mostly consistent with the times shown above for mid data; DuckDB took more than 5 hours to ingest the whole file, followed by PostgreSQL that took a bit less than 3 hours, followed by parquet that took about 1 hour and the fastest is, again, Arrow taking only a bit more than 16 minutes.

<div id="htmlwidget-1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"filter":"none","vertical":false,"fillContainer":false,"data":[["1","2","3","4"],["duckdb_all","pg_all","parquet_all","arrow_all"],[19481.56,9967.75,3916.28,986.55],["5h 24m 41.6s","2h 46m 7.8s","1h 5m 16.3s","16m 26.5s"]],"container":"<table class=\"display cell-border responsive nowrap compact\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>name<\/th>\n      <th>seconds<\/th>\n      <th>time<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"t","columnDefs":[{"visible":false,"targets":2},{"className":"dt-right","targets":2},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data, displayNum, displayIndex, dataIndex) {\nvar value=data[2]; $(this.api().cell(row, 3).node()).css({'background':isNaN(parseFloat(value)) || value <= 0.000000 ? '' : 'linear-gradient(-90.000000deg, transparent ' + Math.max(19481.560000 - value, 0)/19481.560000 * 100 + '%, deepskyblue ' + Math.max(19481.560000 - value, 0)/19481.560000 * 100 + '%)'});\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script>

<br>

<details>
<summary>
Session Info
</summary>

    ## R version 4.1.2 (2021-11-01)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 19043)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_United States.1252 
    ## [2] LC_CTYPE=English_United States.1252   
    ## [3] LC_MONETARY=English_United States.1252
    ## [4] LC_NUMERIC=C                          
    ## [5] LC_TIME=English_United States.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] conflicted_1.0.4  tarchetypes_0.3.2 targets_0.9.0     odbc_1.3.2       
    ##  [5] arrow_6.0.0.2     duckdb_0.3.0      DBI_1.1.1         ggplot2_3.3.5    
    ##  [9] tidyr_1.1.4       dplyr_1.0.7       magrittr_2.0.1   
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.7        compiler_4.1.2    pillar_1.6.4      tools_4.1.2      
    ##  [5] digest_0.6.29     bit_4.0.4         lifecycle_1.0.1   tibble_3.1.6     
    ##  [9] gtable_0.3.0      pkgconfig_2.0.3   rlang_0.4.12      igraph_1.2.10    
    ## [13] rstudioapi_0.13   cli_3.1.0         yaml_2.2.1        xfun_0.29        
    ## [17] fastmap_1.1.0     knitr_1.37        withr_2.4.3       fs_1.5.2         
    ## [21] generics_0.1.1    vctrs_0.3.8       hms_1.1.1         bit64_4.0.5      
    ## [25] grid_4.1.2        tidyselect_1.1.1  data.table_1.14.2 glue_1.6.0       
    ## [29] R6_2.5.1          processx_3.5.2    fansi_0.5.0       pacman_0.5.1     
    ## [33] purrr_0.3.4       callr_3.7.0       blob_1.2.2        codetools_0.2-18 
    ## [37] scales_1.1.1      ps_1.6.0          ellipsis_0.3.2    assertthat_0.2.1 
    ## [41] colorspace_2.0-2  utf8_1.2.2        munsell_0.5.0     cachem_1.0.6     
    ## [45] crayon_1.4.2

</details>

# Wrap-up

My take home messages

-   Arrow is indeed (unbelievably) fast to parse a csv and ingest data,
    particularly, into the arrow file format.
-   DuckDB looked fine at the beginning, but got slow when the data got larger.
    Would need to check why that happens (it should be faster than Postgres).
-   Despite being fast, Arrow did struggle with encoding.
-   I could not quickly find how to tell arrow to read/ingest just some rows and
    not the whole csv file. It seems there is only support for `skip_rows` before column names and `skip_rows_after_names`, but not something like `max_rows`. This would be handy to inspect huge files and maybe write and test queries on a small sample, before send it on the whole data. In DuckDB, even though `read_csv()` does not have such an option, you can work around this by using a LIMIT statement while querying the csv file ([but watch out if you are trying to avoid invalid lines, that approach can bite you](https://github.com/duckdb/duckdb/issues/2777)).

Some non-exaustive TODOs:

-   [ ] Try to find the underlying cause of DuckDB getting slower
-   [ ] Perhaps fiddle with config options for each backend and ingest method. So far I used only the defaults.
-   [ ] Compare size on disk
-   [ ] Compare aggregate queries
-   [ ] Compare joins
-   [ ] Sometime, try and run this kind of thing on a not-so-poorman’s laptop
-   [ ] …

[^1]: meaning it does not fit comfortably in my laptop’s 32GB of RAM.

[^2]: [Vaex](https://vaex.io/docs/index.html) has also been in my
    watch-list, and include most of these features. But it is Python-centric.

[^3]: So this is not really a formal benchmark. Both DuckDB and Arrow
    have extensive testing and routinely benchmark their software, and this exercise by no means aims to oppose or replace that.

[^4]: The whole data comprise 12 csv files, each with 400M - 900M rows and uncompressed size between 50GB and 140GB.

[^5]: No `bench::mark()` here, though, ’cause it would have taken \~4-5
    days to run.
