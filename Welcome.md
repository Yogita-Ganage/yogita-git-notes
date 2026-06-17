users_file_path = "Files/landing/DRJ/Users/2026/06/16/Users_20260616160204.csv"

df_users_file = spark.read.options(
    header=True,
    delimiter=",",
    inferSchema=True
).csv(users_file_path)

sample_df = df_users_file.limit(100)

sample_df.write.format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("test_bronze_drj_users_sample_100")





DESCRIBE test_bronze_drj_users_sample_100;



SELECT episodeNumber
FROM test_bronze_drj_users_sample_100
WHERE episodeNumber IS NOT NULL
LIMIT 20;