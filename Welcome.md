config_df = spark.read.options(header=True, delimiter=",").csv("Files/bronze_Control_file_prod.csv")

config_df.printSchema()
display(config_df.limit(20))