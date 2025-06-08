
# Scala for Data Engineering Tools

- 1. Core Concepts
  - Resilient Distributed Dataset (RDD) ----> Immutable ----> Nothing Overwritten (only copies created)
    - RDDs ----> Distributed --split_across--> Cluster ----> Parallel Processing
      - Distributed RDDs ----> Fault Tolerant ----> Transformed Parition Lost (Not Source) --spark_recovers--> Knowing Transformations Lineage
      - Less Data Moving ----> Process --same--> Cluster (data resides here)

  | Operation                  | Syntax                                                | Description                            | CPG Promo Use Case                                                            |
  | -------------------------- | ----------------------------------------------------- | -------------------------------------- | ----------------------------------------------------------------------------- |
  | **`map`**                  | `rdd.map(x => x.toUpperCase)`                         | Transforms each element                | Convert product names to uppercase for standardization                        |
  | **`filter`**               | `rdd.filter(x => x.contains("PROMO"))`                | Selects elements that meet a condition | Filter only promotional transactions                                          |
  | **`flatMap`**              | `rdd.flatMap(line => line.split(","))`                | Flattens the results of map            | Split promo tags or keywords from a CSV field                                 |
  | **`reduce`**               | `rdd.reduce((a, b) => a + b)`                         | Aggregates elements using a function   | Sum total units sold across promos                                            |
  | **`reduceByKey`**          | `rdd.reduceByKey(_ + _)`                              | Aggregate values by key                | Sum sales by product ID                                                       |
  | **`groupByKey`**           | `rdd.groupByKey()`                                    | Groups values by key (less efficient)  | Group all promo discounts by product ID                                       |
  | **`aggregateByKey`**       | `rdd.aggregateByKey(0)(_ + _, _ + _)`                 | Advanced aggregation                   | Compute total and average sales per product                                   |
  | **`union`**                | `rdd1.union(rdd2)`                                    | Combines two RDDs                      | Combine online and offline promo datasets                                     |
  | **`join`**                 | `rdd1.join(rdd2)`                                     | Join two RDDs by key                   | Join promo sales with product master data                                     |
  | **`leftOuterJoin`**        | `rdd1.leftOuterJoin(rdd2)`                            | Keep all keys from left RDD            | Ensure all promo transactions are retained, even without matching master data |
  | **`rightOuterJoin`**       | `rdd1.rightOuterJoin(rdd2)`                           | Keep all keys from right RDD           | All product IDs from master appear even if no promo                           |
  | **`cartesian`**            | `rdd1.cartesian(rdd2)`                                | All pairwise combinations              | Rarely used; combine promos with all stores for simulation                    |
  | **`distinct`**             | `rdd.distinct()`                                      | Removes duplicates                     | Get unique promo codes                                                        |
  | **`sample`**               | `rdd.sample(withReplacement = false, fraction = 0.1)` | Random sampling                        | Test promo performance on 10% of transactions                                 |
  | **`take(n)`**              | `rdd.take(5)`                                         | Returns first n elements               | Preview first few promo records                                               |
  | **`collect()`**            | `rdd.collect()`                                       | Return all data to driver              | Avoid for large datasets                                                      |
  | **`count()`**              | `rdd.count()`                                         | Count total records                    | Count number of promo transactions                                            |
  | **`saveAsTextFile(path)`** | `rdd.saveAsTextFile("s3://path/")`                    | Save RDD output                        | Export processed promo data                                                   |


  - Transformation Example
  ```
  import org.apache.spark.sql.SparkSession
  import org.apache.spark.sql.functions._
  
  val spark = SparkSession.builder()
    .appName("Promo Data Pipeline")
    .getOrCreate()
  
  // Read promo data as DataFrame
  val df = spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv("s3://bucket/promo_data.csv")
  
  // Filter only active promos
  val df_promo = df.filter(col("promo_status") === "ACTIVE")
  
  // Select product_id and units_sold
  val df_pairs = df_promo.select("product_id", "units_sold")
  
  // Aggregate total units sold by product_id
  val df_total = df_pairs.groupBy("product_id")
    .agg(sum("units_sold").alias("total_units_sold"))
  
  // Show the result
  df_total.show()
  ```

  - Lazy Transformations
    - Transformations --like--> map, filter, groupBy, join, withColumn --are--> Lazy --actions_trigger--> show(), collect(), count(), write()   

- Spark Stream (Optional for now)
  ```
  val streamDF = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host:port")
  .option("subscribe", "promo_topic")
  .load()
  ```

- Data Source Integrations 
  HDFS: "hdfs://path/to/file"
  S3: "s3a://bucket/path/"  
  Kafka: via spark-sql-kafka connector
  JDBC: for connecting to RDBMS
  Data Lake : // abfss://<container>@<storage-account>.dfs.core.windows.net/<folder>/promo_data.csv

- Performance Optimization
  repartition() --usage--> rare --only_before--> Heavy Transformations --like--> .agg, .join etc..
    - when advised --distribution--> skewed or uneven --some_nodes--> bottlenecked
    - repartition() --scope_only--> Custom Script (Spark env) --so_no_need--> Remove Repartition

  ```
  val df = spark.read.parquet("s3://bucket/promo_data.parquet")

  // Repartition by product_id for balanced parallel aggregation
  val df_repart = df.repartition(col("product_id"))
  
  val df_agg = df_repart.groupBy("product_id").sum("units_sold")
  df_agg.show()
  ```

  - Avoid Shuffles (Wide vs Narrow)
    - Wide transformations --data_must--> Move --across--> Clusters --highly--> Expensive
    - Narrow transformations --data_stays--> Cluster (No movement needed) ----> Cheap    
    - Wide Transformations Alternatives

    | Wide Transformation  | Description                           | Narrow Alternative(s)                                     | Notes/When to Use Alternative                                                       | Same Result?                |
    | -------------------- | ------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------- |
    | **groupByKey()**     | Group all values by key (shuffle)     | **reduceByKey()**                                         | `reduceByKey` combines map & reduce, less shuffle; preferred for aggregations       | ✅ Yes (for aggregations)    |
    | **reduceByKey()**    | Aggregate by key (shuffle)            | N/A                                                       | Aggregation requires shuffle, no narrow alternative                                 | ✅ Yes                       |
    | **aggregateByKey()** | Aggregate with custom functions       | N/A                                                       | More control than `reduceByKey`, but still a wide transformation                    | ✅ Yes                       |
    | **join()**           | Join two RDDs by key (shuffle)        | **Broadcast join** (if one side is small)                 | Use broadcast join to avoid shuffle when one dataset fits in memory                 | ✅ Yes (same join result)    |
    | **distinct()**       | Remove duplicates (shuffle)           | `.map(x => (x, null)).reduceByKey((_, _) => null).keys`   | Works if you're deduplicating on known keys only                                    | ⚠️ Sometimes (key-based)    |
    | **repartition()**    | Reshuffle data to new partition count | **coalesce()** (without shuffle)                          | Use `coalesce` to reduce partitions; doesn't balance data evenly like `repartition` | ✅ Data same, layout differs |
    | **sortBy()**         | Sort data globally (shuffle)          | Sort locally using `.mapPartitions(_.toList.sortBy(...))` | Only use local sort if global order isn’t needed                                    | ❌ No (only local sort)      |
    | **cartesian()**      | Cartesian product (shuffle)           | N/A                                                       | Extremely expensive; no workaround                                                  | ❌ No                        |


    - Broadcast Join
      - Broadcasting Join --broadcasts--> Smaller Table ----> Across Clusters (All)       
        - Large Fact Table --joins--> Small Dimension Table --prefer--> Join Broadcasting
        - Broadcasting --mostly_happens--> Auto ----> file size < 10 Mb
      
      ```
      import org.apache.spark.sql.functions.broadcast
      
      val df_productDim = spark.read.parquet("s3://bucket/product_catalog.parquet")
      val df_promoSales = spark.read.parquet("s3://bucket/promo_sales.parquet")
      
      val df_joined = df_promoSales.join(broadcast(df_productDim), Seq("product_id")) // broadcasts df_productDim (explicitly)
      
      df_joined.show()
      ```

  
    - Caching --stores--> intermediate results --optimal_when--> df Reusing
    
    ```
    import org.apache.spark.sql.functions.col
    
    val df_promo = spark.read.parquet("s3://bucket/promo_data.parquet")
    
    val df_activePromos = df_promo.filter(col("status") === "ACTIVE").cache()
    
    df_activePromos.count()  // triggers caching
    df_activePromos.show()   // faster, uses cached data
    ```

Below is Pending, not priority for now

* Build ETL Pipelines

  * Use Scala with Spark to build scalable ETL pipelines.
  * Work with structured and semi-structured data (JSON, Parquet, Avro).
  * Understand schema evolution and data serialization formats.
  * Big Data Ecosystem Integration

* Explore how Scala integrates with Hadoop ecosystem.
  * Learn to use Apache Kafka for streaming with Scala.
  * Explore Apache Flink or Akka Streams (optional but useful).
