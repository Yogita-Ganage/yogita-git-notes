from pyspark.sql.functions import lower, col

display(
    config_df.filter(
        (lower(col("source_system")) == "drj") |
        (lower(col("tablename")).contains("user"))
    )
)
