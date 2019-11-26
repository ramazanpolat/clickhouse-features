# ClickHouse Features List
A brief feature list of ClickHouse

# Encoding
Column values can be encoded to save space and optimize queries.

6 types of encoding:
* **Enums**: A type of column that values are mapped to numbers
* **LowCardinality**: Good for string values up to 10K
* **Delta**: Difference between consecutive values
* **DoubleDelta**: Difference between consecutive deltas, good for slowly changing sequences
* **Gorilla**: Efficient for values that does not change often
* **T64**: Strips lower and higher bits that does not change, good for big numbers in a small range


# Compression

2 types of compression algorithms:
* **ZSTD**: Faster compression but lower compression ratio
* **LZ4**: Slower compression but higher compression ratio

# Table Engines

* **MergeTree**: Main table engine
* **Replicated[x]**: ReplicatedMergeTree, ReplicatedSummingMergeTree, ReplicatedReplacingMergeTree, ...
* **ReplacingMergeTree**: ReplacingMergeTree replaced old rows with new ones based on PRIMARY KEY (or sorting key)
* **SummingMergeTree**: Replaces all the rows with the same primary key with one row which contains summarized values
* **AggregatingMergeTree**: Replaces all rows with the same primary key with a single row that stores a combination of states of aggregate functions
* **CollapsingMergeTree**: Asynchronously deletes (collapses) pairs of rows if all of the fields in a sorting key (ORDER BY) are equivalent excepting the particular field Sign which can have 1 and -1 values.
* **VersionedCollapsingMergeTree**: Allows quick writing of object states that are continually changing and deletes old object states in the background. This significantly reduces the volume of storage
* **GraphiteMergeTree**: This engine is designed for thinning and aggregating/averaging (rollup) Graphite data
* **TinyLog**: Typically used with the write-once method: write data one time, then read it as many times as necessary. For example, you can use TinyLog-type tables for intermediary data that is processed in small batches. Note that storing data in a large number of small tables is inefficient.
* **Log**: Differs from TinyLog in that a small file of "marks" resides with the column files. These marks are written on every data block and contain offsets that indicate where to start reading the file in order to skip the specified number of rows
* **StripeLog**: When you need to write many tables with a small amount of data
* **Kafka**: Integrates into Kafka. Publish or subscribe to data flows, Organize fault-tolerant storage, Process streams as they become available
* **HDFS**: Integration with Apache Hadoop ecosystem by allowing to manage data on HDFS via ClickHouse
* **JDBC**: Connect to any JDBC database
* **ODBC**: Connect to any ODBC database
* **remote**: Connect to remote ClickHouse databases
* **URL**: Connect to HTTP Web Servers (possible producing JSON)
* **MySQL**: Connect to MySQL
* **Lazy**: Keeps tables in RAM only
* **Distributed**: This engine does not store data itself, but allows distributed query processing on multiple servers. Reading is automatically parallelized
* **Dictionary**: This engine displays the dictionary data as a ClickHouse table
* **File**: This engine keeps the data in a file in one of the supported
* **Null**: When writing to a Null table, data is ignored. When reading from a Null table, the response is empty
* **View**: Obvious
* **LiveView**: experimental
* **MaterializedView**: Trigger table for an aggregate query
* **Memory**: This engine stores data in RAM, in uncompressed form. Data is stored in exactly the same form as it is received when read. In other words, reading from this table is completely free. Concurrent data access is synchronized. Locks are short: read and write operations don't block each other. Indexes are not supported. Reading is parallelized
* **Buffer**: Buffers the data to write in RAM, periodically flushing it to another table. During the read operation, data is read from the buffer and the other table simultaneously

# Creating a Table Based On Select
You can create a table without specifying columns individually.

```sql
CREATE TABLE table_name
ENGINE = MergeTree() 
ORDER BY (col1, col2) 
AS 
SELECT * FROM table
```

# Data Format

## Protobuf
* Transparent type conversion
* Supports nested types
* Efficient implementation

## Parquet

## ORC

## Template Format

`Website ${domain:Quoted} is ready`

# Table Functions

## URL

## FILE

## MySQL

## INPUT

?

# Row-Level Security

```xml
<users>
  <user_name>
    <databases>
      <database_name>
        <table_name>
          <filter> id = 1 </filter>
        </table_name>
      </database_name>
    </databases>
  </user_name>
</users>
```
# Optimization of queries with ORDER BY

```
:) set optimize_read_in_order = 1


:) 
```

# System Introspection

* system.query_log
* system.query_thread_log
* system.part_log
* system.trace_log
* system.text_log
* system.metric_log

# Constraints in INSERT queries

```sql
CREATE TABLE hits(
  URL String,
  Domain String,
  CONSTRAINT c_valid_url CHECK expression?
)
```

# Queries with Parameters(Prepared queries)

```sql
SELECT count()
FROM test.hits
WHERE CounterID = {id:UInt32}
AND SearchPhrase = {phrase:String}
```

# Data Gaps Filling

```sql
SELECT EventDate, count() FROM table
GROUP BY EventDate ORDER BY EventDate WITH FILL
```

WITH FILL FROM start

WITH FILL FROM start TO end

WITH FILL FROM start TO end STEP step

# TTL expressions

## TTL for columns

```sql
CREATE 
```

## TTL for tables

```sql
CREATE TABLE
```

## Tiered Storage

```xml
<disks>
  <fast_disk>
    <path>/mnt/disk2</path>
  </fast_disk>
</disks>
```

```
ENGINE = MergeTree
ORDEr BY ...
SETTINGS storage_policy = 'ssd_and_hdd'
```

# Tips and Tricks

## Sampling

You can use SAMPLE keyword to sample data automatically. But if you want to do in manually, you can use `rand()` like this:

```sql
SELECT * 
FROM table
WHERE (rand() % 1000) = 0
```

This will sample 1/1000 of actual data.

The problem is, it is not deterministic. Since it uses `rand()`, the resultset will always change.

So a better solution is to JOIN `system.numbers` or just `numbers(n)`:

```sql
SELECT nums.number AS num, *
FROM table
LEFT JOIN system.numbers nums
WHERE (num % 1000 = 0)
```

## Spill to Disk

When the memory is not enough for a huge group by query, the query gets interrupted with an error like this: `Memory limit (for query) exceeded`

In this case, `max_bytes_before_external_group_by` can be used.

**Usage:**

For session only, use SQL command: `set max_bytes_before_external_group_by = 8000000000`

*Note: 1 with 0 zeros is 1000000000 and it means 1 GB*

You also should set `max_memory_usage` at least twice the size of `max_bytes_before_external_group_by`. This is necessary because there are two stages to aggregation: reading the date and forming intermediate data (1) and merging the intermediate data (2). Dumping data to the file system can only occur during stage 1. If the temporary data wasn't dumped, then stage 2 might require up to the same amount of memory as in stage 1.

This settings also can be specified in `config.xml` like this:

```xml
<?xml version="1.0" ?>
<yandex>
    <profiles>
        <default>
            <max_memory_usage_for_all_queries>101310968832</max_memory_usage_for_all_queries>
            <max_bytes_before_external_group_by>50655484416</max_bytes_before_external_group_by>
            <max_memory_usage>101310968832</max_memory_usage>
        </default>
    </profiles>
</yandex>
```





