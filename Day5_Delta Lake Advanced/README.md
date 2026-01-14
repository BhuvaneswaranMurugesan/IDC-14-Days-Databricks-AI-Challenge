### **DAY 5 (14/01/26) – Delta Lake Advanced**

## Learn:

- Time travel (version history)
- MERGE operations (upserts)
- OPTIMIZE & ZORDER
- VACUUM for cleanup

### 🛠️ Tasks:

1. Implement incremental MERGE
2. Query historical versions
3. Optimize tables
4. Clean old files

### 📝 Practice:

```python
from delta.tables import DeltaTable

# MERGE for incremental updates
deltaTable = DeltaTable.forPath(spark, "/delta/events")
updates = spark.read.csv("/path/to/new_data.csv", header=True, inferSchema=True)

deltaTable.alias("t").merge(
    updates.alias("s"),
    "t.user_session = s.user_session AND t.event_time = s.event_time"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()

# Time travel
v0 = spark.read.format("delta").option("versionAsOf", 0).load("/delta/events")
yesterday = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-01").load("/delta/events")

# Optimize
spark.sql("OPTIMIZE events_table ZORDER BY (event_type, user_id)")
spark.sql("VACUUM events_table RETAIN 168 HOURS")

```

### 🔗 Resources:

- [Time Travel](https://www.databricks.com/blog/2019/02/04/introducing-delta-time-travel-for-large-scale-data-lakes.html)
- [MERGE Guide](https://docs.databricks.com/delta/merge.html)
- YT Resources
    
     1.  [Time Travel](https://youtu.be/0t-GbCW2j24?si=IfrHAfMvLqaqUO-t)
    
    1. [MERGE operations (upserts)](https://youtu.be/xQQGhmsjHv8?si=h9GYnRh6kpD996_C)
    2. [OPTIMIZE & ZORDER](https://youtu.be/KAAyqelKKgw?si=I0-NAwXcWL3bvh8R)
    3. [VACUUM for cleanup](https://youtu.be/k1XHb0kPB4M?si=epCM0bVkgpQtbeBR)