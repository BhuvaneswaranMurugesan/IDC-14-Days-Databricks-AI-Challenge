## DAY 0 – Setup & Data Loading (Prerequisites)

### Overview

Before starting Day 1, complete this setup to load the e-commerce dataset directly from Kaggle into your Databricks workspace.

### **Step 1: Create Databricks Account**

1. Go to [Databricks Community Edition](https://bit.ly/4nK0NTN)
2. Sign up for free account
3. Verify email and log in
4. Create a cluster (use default settings)

### **Step 2: Get Kaggle API Credentials**

1. Go to [Kaggle.com](https://www.kaggle.com/) and log in
2. Click on your profile picture → **Account**
3. Scroll to **API** section → Click **Create New API Token**
4. Download `kaggle.json` (contains your credentials)
5. Open the file and note your `username` and `key`

### **Step 3: Load Data in Databricks**

Create a new notebook in Databricks and run these cells:

### **Notebook:**

### 1. Install Dependencies

```markdown
!pip install kaggle
```

### 2. Configure Kaggle Credentials

```python
import os

os.environ["KAGGLE_USERNAME"] = "your_username"
os.environ["KAGGLE_KEY"] = "your_key"

print("Kaggle credentials configured!")
```

### 3. Create Database Schema

```python
spark.sql("""
CREATE SCHEMA IF NOT EXISTS workspace.ecommerce
""")
```

### 4. Create Volume for Data Storage

```python
spark.sql("""
CREATE VOLUME IF NOT EXISTS workspace.ecommerce.ecommerce_data
""")
```

### 5. Download Dataset from Kaggle

```bash
cd /Volumes/workspace/ecommerce/ecommerce_data
kaggle datasets download -d mkechinov/ecommerce-behavior-data-from-multi-category-store
```

### 6. Extract Downloaded Dataset

```bash
cd /Volumes/workspace/ecommerce/ecommerce_data
unzip -o ecommerce-behavior-data-from-multi-category-store.zip
ls -lh
```

### 7. Clean Up Zip File

```bash
cd /Volumes/workspace/ecommerce/ecommerce_data
rm -f ecommerce-behavior-data-from-multi-category-store.zip
ls -lh
```

### 8. Restart Python Environment

```python
%restart_python
```

### 9. Load November 2019 Data

```python
df_n = spark.read.csv("/Volumes/workspace/ecommerce/ecommerce_data/2019-Nov.csv")
```

### 10. Load October 2019 Data

```python
df = spark.read.csv("/Volumes/workspace/ecommerce/ecommerce_data/2019-Oct.csv")
```

### 11. Display Dataset Statistics and Schema

```python
print(f"October 2019 - Total Events: {df.count():,}")
print("\n" + "="*60)
print("SCHEMA:")
print("="*60)
df.printSchema()
```

### 12. Display Sample Data

```python
print("\n" + "="*60)
print("SAMPLE DATA (First 5 rows):")
print("="*60)
df.show(5, truncate=False)
```

### For better understanding, check the setup guide video: https://youtu.be/nHGMcrxHqrA

---

### **Expected Schema**

After loading, your dataset will have **9 columns**:

| Column | Type | Description | Notes |
| --- | --- | --- | --- |
| event_time | timestamp | When event happened (UTC) | Format: YYYY-MM-DD HH:MM:SS UTC |
| event_type | string | Type of event | Values: view, cart, purchase, remove_from_cart |
| product_id | long | Unique product identifier | Numeric ID |
| category_id | long | Category identifier | Numeric ID |
| category_code | string | Category hierarchy | e.g., "electronics.smartphone" (can be null) |
| brand | string | Product brand | Lowercase (can be null) |
| price | double | Product price in USD | Positive values |
| user_id | long | Permanent user identifier | Numeric ID |
| user_session | string | Session identifier | UUID format, changes per session |

---

### **Dataset Size Reference**

| File | Events | Size | Recommended Use |
| --- | --- | --- | --- |
| **2019-Oct.csv** | ~4.2M | ~1.1 GB | Days 1-3 (learning basics) |
| **2019-Nov.csv** | ~9.3M | ~2.2 GB | Days 4+ (full analysis) |
| **Combined** | ~13.5M | ~3.3 GB | Days 8+ (comprehensive analysis) |

**💡 Pro Tip:** Start with October data for faster iterations, then scale to combined dataset

---

### **Troubleshooting**

- **Issue: Kaggle credentials not working**
    
    ```python
    # Method 1: Set credentials directly
    os.environ['KAGGLE_USERNAME'] = "your_username"
    os.environ['KAGGLE_KEY'] = "your_key"
    
    # Method 2: Verify kaggle config
    !cat ~/.kaggle/kaggle.json
    
    ```
    
- **Issue: File not found in DBFS**
    
    ```python
    # Check DBFS contents
    display(dbutils.fs.ls("/FileStore/ecommerce_data/"))
    
    # Alternative path check
    dbutils.fs.ls("dbfs:/FileStore/ecommerce_data/")
    
    ```
    
- **Issue: Memory errors with large dataset**
    
    ```python
    # Use sampling for testing
    sampled = load_ecommerce_data("Oct", sample_fraction=0.1)
    print(f"Sampled data: {sampled.count():,} rows")
    
    ```
    
- **Issue: Schema inference problems**
    
    ```python
    # Explicitly define schema
    from pyspark.sql.types import StructType, StructField, TimestampType, StringType, LongType, DoubleType
    
    schema = StructType([
        StructField("event_time", TimestampType(), True),
        StructField("event_type", StringType(), True),
        StructField("product_id", LongType(), True),
        StructField("category_id", LongType(), True),
        StructField("category_code", StringType(), True),
        StructField("brand", StringType(), True),
        StructField("price", DoubleType(), True),
        StructField("user_id", LongType(), True),
        StructField("user_session", StringType(), True)
    ])
    
    events = spark.read.csv("/FileStore/ecommerce_data/2019-Oct.csv",
                            header=True, schema=schema)
    
    ```

### **Alternative: Manual Upload (If Kaggle API Doesn't Work)**

1. **Download manually:**
    - Go to [Kaggle Dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)
    - Click "Download" button
    - Extract `2019-Oct.csv` and `2019-Nov.csv`
2. **Upload to Databricks:**
    - Method A: Click **Data** → **Create Table** → Upload files
    - Method B: Drag and drop files into notebook cell
3. **Note the path:**

```python
   # Path will be shown after upload, typically:
   # /FileStore/tables/2019_Oct.csv

   events = spark.read.csv("/FileStore/tables/2019_Oct.csv",
                          header=True, inferSchema=True)

```

---

### ✅ **Setup Complete Checklist**

- [✅]  Databricks account created and cluster running
- [✅]  Kaggle credentials configured
- [✅]  Dataset downloaded (2019-Oct.csv, 2019-Nov.csv)
- [✅]  Files uploaded to DBFS
- [✅]  Data loaded successfully (verified with count)
- [✅]  Schema validated (9 columns confirmed)
- [✅]  Sample queries executed successfully

---

### 🚀 **Ready for Day 1!**

Once all checklist items are complete, you're ready to start **Day 1: Platform Setup & First Steps**

**Quick Start Code for Day 1:**

```python
# Load your data
events = load_ecommerce_data("Oct")

# Verify it's working
print(f"✅ Ready to go! Loaded {events.count():,} events")
events.show(5)

# Your Day 1 challenges start here...

```

---

### 📚 **Additional Resources**

- [Kaggle Dataset Page](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)
- [Databricks Documentation](https://docs.databricks.com/)
- [PySpark DataFrame Guide](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)
- Dataset provided by [REES46 Open CDP](https://rees46.com/en/open-cdp)