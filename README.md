# ClickHouse Features List
A brief feature list of ClickHouse

# Encoding
Column values can be encoded to save space and optimize queries.

6 types of encoding:
* Enums: A type of column that values are mapped to numbers
* LowCardinality: Good for string values up to 10K
* Delta: Difference between consecutive values
* DoubleDelta: Difference between consecutive deltas, good for slowly changing sequences
* Gorilla: Efficient for values that does not change often
* T64: Strips lower and higher bits that does not change, good for big numbers in a small range


# Compression

2 types of compression algorithms:
* **ZSTD**: Faster compression but lower compression ratio
* **LZ4**: Slower compression but higher compression ratio

