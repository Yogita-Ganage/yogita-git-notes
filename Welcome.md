landing_path = "Files/landing/DRJ/Users*.csv"

df_users_landing = spark.read.options(
    header=True,
    delimiter=",",
    inferSchema=True
).csv(landing_path)

print(df_users_landing.columns)