from pyspark.sql.functions import lower, col

drj_users_config = config_df.filter(
    (lower(col("sourcesystem")) == "drj") &
    (lower(col("tablename")) == "users")
)

display(drj_users_config)