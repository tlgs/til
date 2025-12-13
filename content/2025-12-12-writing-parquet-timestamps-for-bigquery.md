+++
title = "Writing Parquet timestamps for BigQuery"
description = "Practical notes on Parquet timestamp encoding from Pandas and how to avoid timezone issues in Google BigQuery."
tags = ["data engineering", "bigquery"]
+++

Some things to keep in mind when working with pandas [`DatetimeIndex`][1]
and looking to persist data using Parquet to back a table in BigQuery:

- timestamps must be physical Parquet columns (index metadata is ignored)
- store UTC instants (convert to UTC, then drop timezone metadata)
- BigQuery `TIMESTAMP` only supports microsecond precision

```python
df.index = df.index.tz_convert(None)
df = df.reset_index()

df.to_parquet(
    buffer,
    engine="pyarrow",
    compression="snappy",
    index=False,
    coerce_timestamps="us",
    allow_truncated_timestamps=True,
    use_deprecated_int96_timestamps=False,
)
```

Without this, timestamps were loaded with the wrong type / incorrect values.

[1]: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DatetimeIndex.html
