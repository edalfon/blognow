---
title: Ingest data to Postres via Pandas
author: edalfon
date: '2017-07-21'
categories:
  - Python
tags:
  - PWeave
---

I have seen and used a few different approaches to copy data from Pandas to
PostgreSQL. So I wanted to leave a few examples here for future reference and
perhaps throw a couple of quick-and-dirty time comparisons/benchmarks.

# The data

For the examples I will use one month of the
[NYC's trip record data.](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
The file was downloaded from
[here](https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2016-12.csv)

```python
import pandas as pd
df = pd.read_csv('/yellow_tripdata_2016-12.csv', nrows=100000)
```

# Pandas' to_sql

Arguably the most straigthforward alternative is to use Pandas' `to_sql`
method.

So you would first use `sqlalchemy` to create and engine and then pass that
connection to pandas.to_sql. Let's see:

```python
# quick and dirty, just hard-code the connection string here
from sqlalchemy import create_engine
engine = create_engine('postgresql://postgres:postgres@localhost:5432/testdb')
```

Now, to_sql has a few arguments we can tweak, so let's try some of those.

## With defaults

```python, echo=False
res = engine.execute('DROP TABLE IF EXISTS df;')
```

```python
%%time
df.to_sql('df', engine, index=False)
```

```python
%%time
df.to_sql('df', engine, if_exists='replace', index=False)
```

That's of course painfully slow because it basically generates a bunch of
INSERT clauses (one per row).

## Method = 'multi'

You can tweak behaviour setting the method argument to 'multi' in order to
"Pass multiple values in a single INSERT clause". This won's help either, and
can actually make things worse.

```python, echo=False
res = engine.execute('DROP TABLE IF EXISTS df;')
```
```python
%%time
df.to_sql('df', engine, if_exists='replace', index=False, method='multi')
```

## Chunk size

For large tables, you may want to set the `chunksize` argument.
Moving the data in chunks could make things slower, but it could help prevent
annoying error (e.g. memory errors, since to_sql need to prepare the insert sql
statements).

```python
%time df.to_sql('df', engine, if_exists='replace', index=False, chunksize=10000)
%time df.to_sql('df', engine, if_exists='replace', index=False, chunksize=100000)
%time df.to_sql('df', engine, if_exists='replace', index=False, chunksize=500000)
%time df.to_sql('df', engine, if_exists='replace', index=False, chunksize=1000000)
%time df.to_sql('df', engine, if_exists='replace', index=False, chunksize=5000000)
```

## Method = callable

You can also pass a function as the method and that enables you, for example,
to use Postgres COPY clause that would be more performant that the INSERTs.

So let's use the function on the sample implementation
[here:](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#io-sql-method)

```python
# Taken as is (i.e. copy-paste) from
# https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#io-sql-method
#
# Alternative to_sql() *method* for DBs that support COPY FROM
import csv
from io import StringIO

def psql_insert_copy(table, conn, keys, data_iter):
    """
    Execute SQL statement inserting data

    Parameters
    ----------
    table : pandas.io.sql.SQLTable
    conn : sqlalchemy.engine.Engine or sqlalchemy.engine.Connection
    keys : list of str
        Column names
    data_iter : Iterable that iterates the values to be inserted
    """
    # gets a DBAPI connection that can provide a cursor
    dbapi_conn = conn.connection
    with dbapi_conn.cursor() as cur:
        s_buf = StringIO()
        writer = csv.writer(s_buf)
        writer.writerows(data_iter)
        s_buf.seek(0)

        columns = ', '.join('"{}"'.format(k) for k in keys)
        if table.schema:
            table_name = '{}.{}'.format(table.schema, table.name)
        else:
            table_name = table.name

        sql = 'COPY {} ({}) FROM STDIN WITH CSV'.format(
            table_name, columns)
        cur.copy_expert(sql=sql, file=s_buf)
```

Then pass that function to `to_sql`

```python, echo=False
res = engine.execute('DROP TABLE IF EXISTS df;')
```
```python
%%time
df.to_sql('df', engine, if_exists='replace', index=False, method=psql_insert_copy)
```

This is clearly much faster.

# copy_from method

Before Pandas included the option to pass a callable as the method for to_sql,
which was only added in Pandas 0.24, I guess the fastest was to use the
copy_from method from the cursor directly (powered by `psycopg2`). See [this]
(https://stackoverflow.com/questions/23103962/how-to-write-dataframe-to-postgres-table)
answer on StackOverflow.

```python
from io import StringIO

# Code adapted from https://stackoverflow.com/questions/23103962/how-to-write-dataframe-to-postgres-table
def pandas_to_postgresql(engine, df, table_name):
    df.head(0).to_sql(table_name, engine, if_exists='replace', index=False)
    conn = engine.raw_connection()
    cur = conn.cursor()
    output = StringIO()
    df.to_csv(output, sep='\t', header=False, index=False)
    output.seek(0)
    cur.copy_from(output, table_name, sep='|', null="") # null values become ''
    conn.commit()
```

```python
%%time
pandas_to_postgresql(engine, df, 'df')
```

Of course it is faster than the default `to_sql`, but a bit slower than
using `to_sql` with the custom method to copy data into PostreSQL.

# PostgreSQL's COPY directly

Ok, the fastest methods above use PostgreSQL's
[COPY FROM command](https://www.postgresql.org/docs/current/sql-copy.html)
that efficiently "copies data from a file to a table". Those take advantage of
the fact that COPY can read the data from STDIN. But you can read the csv
file into PostgreSQL directly, without reading it before into Pandas.

By copying the data directly from the csv file to PostgreSQL you can save some
time (Pandas' reading and parsing time). But that is actually a different use
case that sometimes may not be what you want, and sometimes may not even be
possible. So here some caveats:

- The server must be able to access the file and the account running the server
  must have privileges to read the file. So tipically the file would be stored
  on the same machine running the PostgreSQL server (although you can hack your
  way around to get the server to read a remote file; sometimes using the
  program option could be useful). A typical use case is a server that you
  install in your laptop to process some larguish data (that do not fit in
  memory, but it is not that big to warrant/need distributed computing or more
  powerful hardware).

- You may not want to do it 'cause you want to preprocess the data using,
  for example, Pandas. One of the reasons for this is that, even though it could
  be faster, PostgreSQL (and RDBMS in general) tend to be picky reading the
  data and will simply fail when it finds ill-behaved rows (in Pandas you can
  for example set `error_bad_lines=False` and `warn_bad_lines=True`).


```python
df.head(0).to_sql(table_name, engine, if_exists='replace', index=False)

```

# Session Info
```python, results='verbatim'
from sinfo import sinfo
sinfo_html = sinfo(html=True)
display_markdown(sinfo_html.data, raw=True)
```
