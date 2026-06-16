I checked the DRJ Users column flow today. The episodeNumber column is not available in bronze_drj_users, so I traced it back to the Bronze load layer. I found that DRJ Users is loaded through the Lake To Bronze Table Load notebook using the bronze_Control_file_prod.csv config, and the latest landing file does contain the episodeNumber column. So the source file is getting the column, but it is not currently reflected in the Bronze table schema. I’ll continue the next step tomorrow to check the Bronze schema/load handling and confirm the proper fix.

  
  
  

checklist for tommorow

Option 1: ALTER TABLE bronze_drj_users ADD COLUMN episodeNumber STRING

Option 2: Drop/recreate bronze_drj_users from source

Option 3: Enable schema evolution/mergeSchema in Bronze load logic