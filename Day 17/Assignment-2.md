This assignment involves analyzing movie data using **Apache Spark** and **Hadoop Distributed File System (HDFS)**. The goal is to process and analyze three datasets: **movies.csv**, **ratings.csv**, and **tags.csv**, and answer a series of questions related to movie ratings, tags, and user behavior. Below is a detailed explanation of the assignment:

---

### **Problem Statement**
The assignment requires the following steps:

1. **Setup Hadoop Cluster**: 
   - Set up a Hadoop cluster with YARN, Hive, and Spark using Docker or a cloud platform like AWS, GCP, or Azure.
   - Load the three datasets (`movies.csv`, `ratings.csv`, and `tags.csv`) into HDFS.

2. **Perform Data Analysis**:
   - Write Spark jobs to solve the following problem statements:
     - **a.** Show the aggregated number of ratings per year.
     - **b.** Show the average monthly number of ratings.
     - **c.** Show the rating levels distribution.
     - **d.** Show the 18 movies that are tagged but not rated.
     - **e.** Show the movies that have ratings but no tags.
     - **f.** Focus on the rated but untagged movies with more than 30 user ratings and show the top 10 movies in terms of average rating and number of ratings.
     - **g.** Calculate the average number of tags per movie and per user, and compare them.
     - **h.** Identify users who tagged movies without rating them.
     - **i.** Calculate the average number of ratings per user and per movie.
     - **j.** Determine the predominant (most frequent) genre per rating level.
     - **k.** Find the predominant tag per genre and the most tagged genres.
     - **l.** Identify the most popular movies based on the number of users who rated them.
     - **m.** Find the top 10 movies in terms of average rating (with more than 30 user reviews).

3. **Store Output**:
   - Save the output of each problem statement as a single CSV file with headers in an HDFS output path.

4. **Deliverables**:
   - Provide the final executable Spark code.
   - Add proper documentation for the logic used in each problem statement.
   - Follow best practices for writing Spark code.
   - Attach screenshots of the final output in the documentation.

---

### **Tools and Technologies Used**
The following tools and technologies were used to complete this assignment:

1. **Python3**: The primary programming language used for writing Spark jobs.
2. **Apache Spark**: Used for distributed data processing and analysis.
3. **Google Cloud Platform (GCP)**:
   - **DataProc**: A managed Spark and Hadoop service on GCP.
   - **Google Cloud Storage (GCS)**: Used for storing input and output files.
4. **Jupyter Lab**: Used for writing and executing Spark code interactively.
5. **Apache Hive**: Used for querying and managing structured data in HDFS.
6. **HDFS**: Used for storing and managing the input datasets and output files.

---

### **Process and File Descriptions**

#### **Step 1: Data Ingestion**
- The three CSV files (`movies.csv`, `ratings.csv`, and `tags.csv`) were placed in the HDFS location.
- These files were then ingested into Spark DataFrames:
  - **Movies DataFrame**: Contains movie details like `movieId`, `title`, and `genres`.
  - **Ratings DataFrame**: Contains user ratings for movies, including `userId`, `movieId`, `rating`, and `timestamp`.
  - **Tags DataFrame**: Contains user-generated tags for movies, including `userId`, `movieId`, `tag`, and `timestamp`.

#### **Step 2: Data Analysis Using Spark SQL**
- Temporary views were created for the three DataFrames (`MOVIES`, `RATINGS`, and `TAGS`) to enable SQL-like querying.
- Spark SQL queries were written to solve each problem statement. For example:
  - **Aggregated number of ratings per year**: A query was written to group ratings by year and count the number of ratings.
  - **Average monthly number of ratings**: A query was written to calculate the average number of ratings per month.
  - **Rating levels distribution**: A query was written to categorize ratings into buckets (e.g., 0.0-2.0, 2.5-4.0, >4) and calculate their distribution.
  - **Movies tagged but not rated**: A query was written to find movies that have tags but no ratings.
  - **Movies rated but not tagged**: A query was written to find movies that have ratings but no tags.
  - **Top 10 movies by average rating and number of ratings**: A query was written to find the top 10 movies with more than 30 user ratings, ranked by average rating and number of ratings.

#### **Step 3: Output Storage**
- The output of each query was saved as a single CSV file with headers in the HDFS output path.
- The output files were verified using the HDFS Namenode UI.

---

### **Key Queries and Logic**

1. **Aggregated Number of Ratings Per Year**:
   - Query: Group ratings by year and count the number of ratings.
   - Logic: Extract the year from the `timestamp` column and group by year.

2. **Average Monthly Number of Ratings**:
   - Query: Calculate the average number of ratings per month.
   - Logic: Extract the year and month from the `timestamp` column and group by month.

3. **Rating Levels Distribution**:
   - Query: Categorize ratings into buckets (e.g., 0.0-2.0, 2.5-4.0, >4) and calculate their distribution.
   - Logic: Use a `CASE` statement to categorize ratings and calculate the percentage of each bucket.

4. **Movies Tagged but Not Rated**:
   - Query: Find movies that have tags but no ratings.
   - Logic: Perform a left join between the `TAGS` and `RATINGS` DataFrames and filter out movies with no ratings.

5. **Movies Rated but Not Tagged**:
   - Query: Find movies that have ratings but no tags.
   - Logic: Perform a left join between the `RATINGS` and `TAGS` DataFrames and filter out movies with no tags.

6. **Top 10 Movies by Average Rating and Number of Ratings**:
   - Query: Find the top 10 movies with more than 30 user ratings, ranked by average rating and number of ratings.
   - Logic: Filter movies with more than 30 ratings, calculate the average rating, and rank them.

---

### **Output Files**
The output of each query was saved as a CSV file in the HDFS output path. Some of the output files include:
- `agg_Ratings.csv`: Aggregated number of ratings per year.
- `avg_monthly_Ratings.csv`: Average monthly number of ratings.
- `distribution_ratings.csv`: Rating levels distribution.
- `tagged_not_rated.csv`: Movies tagged but not rated.
- `rated_not_tagged.csv`: Movies rated but not tagged.
- `top_10_avgratings&count_ratings.csv`: Top 10 movies by average rating and number of ratings.

---

### **Conclusion**
This assignment demonstrates the use of **Apache Spark** for distributed data processing and analysis. By leveraging Spark SQL, the assignment efficiently processes large datasets to answer complex questions about movie ratings, tags, and user behavior. The output is stored in HDFS, ensuring scalability and compatibility with big data workflows. The use of cloud platforms like GCP further enhances the scalability and performance of the solution.

<br/>
<br/>

The **`Spark_MovieRating.ipynb`** file is a **Jupyter Notebook** that contains the Python code for performing the movie data analysis using **Apache Spark**. It is structured as a series of code cells, each performing a specific task, such as data ingestion, transformation, analysis, and output generation. Below is a detailed explanation of the notebook's content:

---

### **1. Setting Up the Spark Session**
The notebook begins by setting up a **Spark session**, which is the entry point for using Spark functionalities.

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from datetime import datetime
from pyspark.sql.functions import from_unixtime

# Create Spark session
spark = SparkSession.builder \
    .appName("Spark with Hive") \
    .enableHiveSupport() \
    .getOrCreate()
```

- **SparkSession**: This is the entry point for working with Spark. It allows you to create DataFrames, execute SQL queries, and perform distributed computations.
- **enableHiveSupport()**: This enables Hive support, allowing Spark to interact with Hive tables and use HiveQL for queries.

---

### **2. Loading Data into Spark DataFrames**
The notebook loads the three datasets (`movies.csv`, `ratings.csv`, and `tags.csv`) from HDFS into Spark DataFrames.

#### **Loading Movies Data**
```python
hdfs_path = '/tmp/spark_movie/movies.csv'
df_movies = spark.read.format('csv').option('header', 'true').option('inferSchema', 'true').load(hdfs_path)

# Print schema and sample data
df_movies.printSchema()
df_movies.show(5)
```

- **`spark.read.format('csv')`**: Reads the CSV file from HDFS.
- **`option('header', 'true')`**: Treats the first row as the header.
- **`option('inferSchema', 'true')`**: Automatically infers the data types of columns.
- **`printSchema()`**: Prints the schema of the DataFrame (column names and data types).
- **`show(5)`**: Displays the first 5 rows of the DataFrame.

#### **Loading Ratings Data**
```python
schema = StructType([
    StructField("userId", IntegerType(), True),
    StructField("movieId", IntegerType(), True),
    StructField("rating", FloatType(), True),
    StructField("timestamp", IntegerType(), True),
])
hdfs_path = '/tmp/spark_movie/ratings.csv'
df_ratings = spark.read.format('csv').option('header', 'true').option('inferSchema', 'false').schema(schema).load(hdfs_path)

# Convert timestamp to TimestampType
df_ratings = df_ratings.withColumn("timestamp", from_unixtime("timestamp").cast(TimestampType()))

# Show the DataFrame
df_ratings.show()
```

- **`StructType`**: Defines the schema for the DataFrame explicitly (useful when the schema is known).
- **`from_unixtime`**: Converts the Unix timestamp to a human-readable timestamp.
- **`cast(TimestampType())`**: Casts the column to a timestamp data type.

#### **Loading Tags Data**
```python
schema = StructType([
    StructField("userId", IntegerType(), True),
    StructField("movieId", IntegerType(), True),
    StructField("tag", StringType(), True),
    StructField("timestamp", IntegerType(), True),
])

hdfs_path = '/tmp/spark_movie/tags.csv'
df_tags = spark.read.format('csv').option('header', 'true').option('inferSchema', 'false').schema(schema).load(hdfs_path)

# Convert timestamp to TimestampType
df_tags = df_tags.withColumn("timestamp", from_unixtime("timestamp").cast(TimestampType()))

# Show the DataFrame
df_tags.show()
```

- Similar to the ratings data, the tags data is loaded with a defined schema and the timestamp is converted.

---

### **3. Creating Temporary Views for SQL Queries**
To enable SQL-like querying, the notebook creates temporary views for the DataFrames.

```python
df_movies.createOrReplaceTempView("MOVIES")
df_ratings.createOrReplaceTempView("RATINGS")
df_tags.createOrReplaceTempView("TAGS")
```

- **`createOrReplaceTempView`**: Creates a temporary view of the DataFrame that can be queried using Spark SQL.

---

### **4. Performing Data Analysis**
The notebook contains several queries to analyze the data. Each query is executed using Spark SQL, and the results are saved as CSV files in HDFS.

#### **Example Query: Aggregated Number of Ratings Per Year**
```python
query = """
    Select year(timestamp) as year, count(rating) as ratings 
    from RATINGS 
    group by 1 
    order by year(timestamp) desc
"""
output = spark.sql(query)
output.show()

# Save output to HDFS
output.coalesce(1).write.mode("overwrite").format('csv').option('header', 'true').option('delimiter', ',').save('/tmp/output_data/spark_movie/agg_Ratings.csv')
print("Write Successful")
```

- **`year(timestamp)`**: Extracts the year from the timestamp.
- **`group by 1`**: Groups the data by the first column (year).
- **`coalesce(1)`**: Ensures the output is written to a single file.
- **`write.mode("overwrite")`**: Overwrites the output file if it already exists.

#### **Example Query: Average Monthly Number of Ratings**
```python
query = """
    Select left(timestamp, 7) as year_month, avg(rating) as avg_rating
    from RATINGS 
    group by 1 
    order by left(timestamp, 7) desc
"""
output = spark.sql(query)
output.show()

# Save output to HDFS
output.coalesce(1).write.mode("overwrite").format('csv').option('header', 'true').option('delimiter', ',').save('/tmp/output_data/spark_movie/avg_monthly_Ratings.csv')
print("Write Successful")
```

- **`left(timestamp, 7)`**: Extracts the year and month from the timestamp (e.g., `2023-10`).
- **`avg(rating)`**: Calculates the average rating for each month.

#### **Example Query: Rating Levels Distribution**
```python
query = """
    with t1 as (
        Select rating, 
               case when rating between 0 and 2 THEN '0.0-2.0'
                    WHEN rating between 2.3 and 4 THEN '2.5-4.0'
                    ELSE '>4' END as rating_bucket
        from RATINGS
    ),
    t2 as (
        select rating_bucket, count(rating) as counts
        from t1
        group by 1
        order by 1
    )
    Select rating_bucket, counts, counts * 100 / sum(counts) over () as percentage
    from t2
"""
output = spark.sql(query)
output.show()

# Save output to HDFS
output.coalesce(1).write.mode("overwrite").format('csv').option('header', 'true').option('delimiter', ',').save('/tmp/output_data/spark_movie/distribution_ratings.csv')
print("Write Successful")
```

- **`CASE` statement**: Categorizes ratings into buckets (e.g., 0.0-2.0, 2.5-4.0, >4).
- **`sum(counts) over ()`**: Calculates the total number of ratings for percentage calculation.

---

### **5. Saving Output to HDFS**
After each query, the results are saved as CSV files in HDFS. For example:

```python
output.coalesce(1).write.mode("overwrite").format('csv').option('header', 'true').option('delimiter', ',').save('/tmp/output_data/spark_movie/agg_Ratings.csv')
```

- **`coalesce(1)`**: Ensures the output is written to a single file.
- **`mode("overwrite")`**: Overwrites the file if it already exists.
- **`option('header', 'true')`**: Includes the header in the output file.
- **`option('delimiter', ',')`**: Uses a comma as the delimiter in the CSV file.

---

### **6. Key Features of the Notebook**
- **Interactive Execution**: The notebook allows for interactive execution of code cells, making it easy to test and debug queries.
- **Modular Structure**: Each query is contained in a separate cell, making the notebook modular and easy to follow.
- **Output Visualization**: The `show()` function is used to display the results of each query in a tabular format.
- **Scalability**: The use of Spark ensures that the analysis can scale to handle large datasets.

---

### **Conclusion**
The **`Spark_MovieRating.ipynb`** notebook is a comprehensive example of how to use **Apache Spark** for data analysis. It demonstrates:
- Data ingestion from HDFS.
- Schema definition and data transformation.
- SQL-like querying using Spark SQL.
- Saving results to HDFS for further use.

This notebook serves as a practical guide for performing big data analysis using Spark and can be extended to handle more complex datasets and queries.

<br/>
<br/>

Below is a detailed explanation of each question using **PySpark**, along with the corresponding code snippets for solving each problem:

---

### **a. Show the Aggregated Number of Ratings Per Year**
**Objective**: Calculate the total number of ratings given each year.

#### **Explanation**:
- Extract the year from the `timestamp` column.
- Group the data by year and count the number of ratings.

#### **PySpark Code**:
```python
query = """
    Select year(timestamp) as year, count(rating) as ratings 
    from RATINGS 
    group by 1 
    order by year(timestamp) desc
"""
output = spark.sql(query)
output.show()
```

- **`year(timestamp)`**: Extracts the year from the timestamp.
- **`group by 1`**: Groups the data by the first column (year).
- **`order by year(timestamp) desc`**: Orders the results by year in descending order.

---

### **b. Show the Average Monthly Number of Ratings**
**Objective**: Calculate the average number of ratings per month.

#### **Explanation**:
- Extract the year and month from the `timestamp` column.
- Group the data by year and month and calculate the average number of ratings.

#### **PySpark Code**:
```python
query = """
    Select left(timestamp, 7) as year_month, avg(rating) as avg_rating
    from RATINGS 
    group by 1 
    order by left(timestamp, 7) desc
"""
output = spark.sql(query)
output.show()
```

- **`left(timestamp, 7)`**: Extracts the year and month (e.g., `2023-10`).
- **`avg(rating)`**: Calculates the average rating for each month.

---

### **c. Show the Rating Levels Distribution**
**Objective**: Categorize ratings into buckets and calculate their distribution.

#### **Explanation**:
- Use a `CASE` statement to categorize ratings into buckets (e.g., 0.0-2.0, 2.5-4.0, >4).
- Calculate the percentage of ratings in each bucket.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select rating, 
               case when rating between 0 and 2 THEN '0.0-2.0'
                    WHEN rating between 2.3 and 4 THEN '2.5-4.0'
                    ELSE '>4' END as rating_bucket
        from RATINGS
    ),
    t2 as (
        select rating_bucket, count(rating) as counts
        from t1
        group by 1
        order by 1
    )
    Select rating_bucket, counts, counts * 100 / sum(counts) over () as percentage
    from t2
"""
output = spark.sql(query)
output.show()
```

- **`CASE` statement**: Categorizes ratings into buckets.
- **`sum(counts) over ()`**: Calculates the total number of ratings for percentage calculation.

---

### **d. Show the 18 Movies That Are Tagged but Not Rated**
**Objective**: Find movies that have tags but no ratings.

#### **Explanation**:
- Perform a left join between the `TAGS` and `RATINGS` DataFrames.
- Filter out movies with no ratings.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select distinct t.movieID 
        from TAGS as t
        left join RATINGS as r
        on t.movieID = r.movieID
        where r.movieID IS NULL
    )
    Select m.title 
    from MOVIES as m
    inner join t1
    on m.movieID = t1.movieID
    order by 1
"""
output = spark.sql(query)
output.show(18)
```

- **`left join`**: Joins `TAGS` and `RATINGS` to find movies with tags but no ratings.
- **`IS NULL`**: Filters out movies with no ratings.

---

### **e. Show the Movies That Have Ratings but No Tags**
**Objective**: Find movies that have ratings but no tags.

#### **Explanation**:
- Perform a left join between the `RATINGS` and `TAGS` DataFrames.
- Filter out movies with no tags.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select distinct r.movieID 
        from RATINGS as r
        left join TAGS as t
        on t.movieID = r.movieID
        where t.movieID IS NULL
    )
    Select m.title 
    from MOVIES as m
    inner join t1
    on m.movieID = t1.movieID
    order by 1
"""
output = spark.sql(query)
output.show()
```

- **`left join`**: Joins `RATINGS` and `TAGS` to find movies with ratings but no tags.
- **`IS NULL`**: Filters out movies with no tags.

---

### **f. Focus on the Rated but Untagged Movies with More Than 30 User Ratings and Show the Top 10 Movies in Terms of Average Rating and Number of Ratings**
**Objective**: Find the top 10 movies with more than 30 user ratings and no tags, ranked by average rating and number of ratings.

#### **Explanation**:
- Filter movies with more than 30 ratings and no tags.
- Calculate the average rating and number of ratings for each movie.
- Rank the movies by average rating and number of ratings.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select movieid
        from ratings
        group by 1
        having count(distinct userid) > 30
    ),
    t2 as (
        Select t1.movieID 
        from t1
        left join TAGS as t
        on t1.movieID = t.movieID
        where t.movieID IS NULL
    ),
    t3 as (
        Select m.title, m.movieID 
        from MOVIES as m
        inner join t2
        on m.movieID = t2.movieID
        order by 1
    ),
    t4 as (
        Select t3.title, avg(r.rating) as avg_rating,
        dense_rank() over (order by avg(r.rating) desc) as avg_rank
        from t3 
        left join RATINGS as r
        on t3.movieID = r.movieID
        group by 1
    ),
    t5 as (
        Select t3.title, count(rating) as counts,
        dense_rank() over (order by count(rating) desc) as count_rank
        from t3 
        left join RATINGS as r
        on t3.movieID = r.movieID
        group by 1
    )
    Select t4.title as Movie_title1, t4.avg_rank, round(t4.avg_rating, 4) as avg_rating,
           t5.title as Movie_title2, t5.count_rank, t5.counts
    from t4 
    inner join t5
    on t4.avg_rank = t5.count_rank
    where t4.avg_rank <= 10 and t5.count_rank <= 10
"""
output = spark.sql(query)
output.show()
```

- **`having count(distinct userid) > 30`**: Filters movies with more than 30 user ratings.
- **`dense_rank()`**: Ranks movies by average rating and number of ratings.

---

### **g. Calculate the Average Number of Tags Per Movie and Per User, and Compare Them**
**Objective**: Calculate the average number of tags per movie and per user, and compare the two.

#### **Explanation**:
- Calculate the total number of tags divided by the number of distinct movies.
- Calculate the total number of tags divided by the number of distinct users.
- Compare the two averages.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select '1' as key, 
               round((sum(CASE when tag IS NOT NULL THEN 1 ELSE 0 END) / count(distinct movieid)), 2) as tags_per_movie
        from TAGS
    ),
    t2 as (
        Select '1' as key, 
               (sum(CASE WHEN tag IS NOT NULL THEN 1 ELSE 0 END) / count(distinct userid)) as tags_per_user
        from TAGS
    )
    Select t1.tags_per_movie, t2.tags_per_user,
           CASE WHEN tags_per_user > tags_per_movie THEN 'tags_per_user is higher'
                ELSE 'tags_per_movie is higher' END as Comparison
    from t1 
    inner join t2 
    on t1.key = t2.key
"""
output = spark.sql(query)
output.show()
```

- **`sum(CASE when tag IS NOT NULL THEN 1 ELSE 0 END)`**: Counts the number of tags.
- **`count(distinct movieid)`**: Counts the number of distinct movies.
- **`count(distinct userid)`**: Counts the number of distinct users.

---

### **h. Identify Users Who Tagged Movies Without Rating Them**
**Objective**: Find users who tagged movies but did not rate them.

#### **Explanation**:
- Perform a left join between the `TAGS` and `RATINGS` DataFrames.
- Filter out users who did not rate movies.

#### **PySpark Code**:
```python
query = """
    Select distinct t.userid
    from TAGS as t
    left join RATINGS as r
    on t.movieID = r.movieID
    where r.userID is NULL
"""
output = spark.sql(query)
output.show()
```

- **`left join`**: Joins `TAGS` and `RATINGS` to find users who tagged movies but did not rate them.
- **`IS NULL`**: Filters out users who did not rate movies.

---

### **i. Calculate the Average Number of Ratings Per User and Per Movie**
**Objective**: Calculate the average number of ratings per user and per movie.

#### **Explanation**:
- Calculate the total number of ratings divided by the number of distinct users.
- Calculate the total number of ratings divided by the number of distinct movies.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select '1' as key, 
               round((sum(CASE when rating IS NOT NULL THEN 1 ELSE 0 END) / count(distinct userid)), 2) as ratings_per_user
        from RATINGS
    ),
    t2 as (
        Select '1' as key, 
               round((sum(CASE WHEN rating IS NOT NULL THEN 1 ELSE 0 END) / count(distinct movieid)), 2) as ratings_per_movie
        from RATINGS
    )
    Select t1.ratings_per_user, t2.ratings_per_movie
    from t1 
    inner join t2 
    on t1.key = t2.key
"""
output = spark.sql(query)
output.show()
```

- **`sum(CASE when rating IS NOT NULL THEN 1 ELSE 0 END)`**: Counts the number of ratings.
- **`count(distinct userid)`**: Counts the number of distinct users.
- **`count(distinct movieid)`**: Counts the number of distinct movies.

---

### **j. Determine the Predominant (Most Frequent) Genre Per Rating Level**
**Objective**: Find the most frequent genre for each rating level.

#### **Explanation**:
- Group the data by rating and genre.
- Use `dense_rank()` to rank genres by frequency for each rating level.
- Select the top genre for each rating level.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select r.rating, m.genres, count(*) as counts,
        dense_rank() over (partition by r.rating order by count(*) desc) as ranker
        from RATINGS as r
        left join MOVIES as m
        on r.movieID = m.movieID
        group by 1, 2
    )
    Select rating, genres as most_frequent_genre 
    from t1 
    where ranker = 1
    order by rating desc
"""
output = spark.sql(query)
output.show()
```

- **`dense_rank()`**: Ranks genres by frequency for each rating level.
- **`ranker = 1`**: Selects the top genre for each rating level.

---

### **k. Find the Predominant Tag Per Genre and the Most Tagged Genres**
**Objective**: Find the most frequent tag for each genre and the most tagged genres.

#### **Explanation**:
- Group the data by genre and tag.
- Use `dense_rank()` to rank tags by frequency for each genre.
- Select the top tag for each genre.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select m.genres, t.tag, count(*) as counts,
        dense_rank() over (partition by m.genres order by count(*) desc) as ranker
        from MOVIES as m
        left join TAGS as t
        on t.movieID = m.movieID
        group by 1, 2
    )
    Select genres, tag as most_frequent_tag 
    from t1 
    where ranker = 1
    order by genres desc
"""
output = spark.sql(query)
output.show()
```

- **`dense_rank()`**: Ranks tags by frequency for each genre.
- **`ranker = 1`**: Selects the top tag for each genre.

---

### **l. Identify the Most Popular Movies Based on the Number of Users Who Rated Them**
**Objective**: Find the most popular movies based on the number of users who rated them.

#### **Explanation**:
- Group the data by movie and count the number of distinct users who rated each movie.
- Rank the movies by the number of users.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select r.movieID, m.title, count(distinct r.userID) as counts,
        dense_rank() over (order by count(distinct r.userid) desc) as ranker
        from RATINGS as r
        left join MOVIES as m
        on r.movieID = m.movieID
        group by 1, 2
    )
    Select title, counts 
    from t1 
    where ranker <= 10
"""
output = spark.sql(query)
output.show()
```

- **`count(distinct r.userID)`**: Counts the number of distinct users who rated each movie.
- **`dense_rank()`**: Ranks movies by the number of users.

---

### **m. Find the Top 10 Movies in Terms of Average Rating (with More Than 30 User Reviews)**
**Objective**: Find the top 10 movies with more than 30 user reviews, ranked by average rating.

#### **Explanation**:
- Filter movies with more than 30 user reviews.
- Calculate the average rating for each movie.
- Rank the movies by average rating.

#### **PySpark Code**:
```python
query = """
    with t1 as (
        Select movieid, avg(rating) as avg_rating,
        dense_rank() over (order by avg(rating) desc) as ranker
        from RATINGS
        group by 1
        having count(distinct userID) > 30
    )
    Select m.title, round(t1.avg_rating, 9) as avg_rating, t1.ranker 
    from t1
    left join MOVIES as m
    on t1.movieID = m.movieID
    where ranker <= 10
"""
output = spark.sql(query)
output.show()
```

- **`having count(distinct userID) > 30`**: Filters movies with more than 30 user reviews.
- **`dense_rank()`**: Ranks movies by average rating.

---

### **Conclusion**
This PySpark script provides a comprehensive solution for analyzing movie data. It demonstrates how to:
- Load and process data from HDFS.
- Perform complex transformations and aggregations.
- Use Spark SQL for querying and analysis.
- Save results back to HDFS.

Each query is modular and can be extended or modified to suit specific requirements.