# News API Data Pipeline Project Explanation

This project is a comprehensive data pipeline that fetches news articles from the NewsAPI, processes them, stores them in Google Cloud Storage (GCS), and then loads them into Snowflake for analysis. Here's a detailed breakdown of each component:

## 1. Data Fetching (`fetch_news.py`)

This Python script handles the extraction of news data from NewsAPI and its transformation:

### Key Features:
- **API Connection**: Uses NewsAPI's "everything" endpoint to fetch articles about "apple" (could be parameterized)
- **Date Handling**: Gets news from the previous day to current date
- **Data Processing**:
  - Extracts key fields like title, publication date, URL, source, author, and image URL
  - Trims article content to 200 characters or to the last full sentence
- **Storage**:
  - Saves data as a Parquet file (efficient columnar storage format)
  - Uploads to Google Cloud Storage bucket `snowflake_projects` under `news_data_analysis/parquet_files/`
  - Cleans up by removing the local file after upload

### Technical Details:
- Uses `pandas` for data manipulation
- Leverages `google-cloud-storage` for GCS uploads
- Generates unique filenames with timestamps to avoid conflicts

## 2. Snowflake Infrastructure (`snowflake_commands.sql`)

This file sets up the Snowflake environment for receiving the news data:

### Key Components:
1. **Database Creation**: Creates a `news_api` database
2. **File Format**: Defines a Parquet file format for Snowflake to understand the incoming data
3. **Storage Integration**: 
   - Creates a GCS integration named `news_data_gcs_integration`
   - Configures access to the GCS bucket where Parquet files are stored
4. **External Stage**: 
   - Creates a stage `gcs_raw_data_stage` pointing to the GCS location
   - Associates it with the storage integration and file format

## 3. Airflow Orchestration (`airflow_job.py`)

This DAG coordinates the entire workflow using Apache Airflow:

### Pipeline Structure:
1. **Fetch News Data Task**:
   - PythonOperator that runs the `fetch_news_data()` function
   - Executes daily starting August 3, 2024

2. **Snowflake Table Creation Task**:
   - Uses SnowflakeOperator to create a table `news_api_data`
   - Leverages Snowflake's schema inference capability to automatically detect the schema from the Parquet files

3. **Data Loading Task**:
   - Copies data from the GCS stage into the Snowflake table
   - Uses case-insensitive column name matching
   - Specifies the Parquet file format

### DAG Configuration:
- Runs daily (`schedule_interval=timedelta(days=1)`)
- No catchup to avoid backfilling (`catchup=False`)
- Simple error handling with 1 retry after 5 minutes

## Workflow Summary

The complete data flow is:
1. Airflow triggers the DAG daily
2. Python script fetches news data from NewsAPI
3. Data is processed and saved as Parquet to GCS
4. Snowflake table is created (if not exists) with inferred schema
5. Data is loaded from GCS into Snowflake table

## Potential Enhancements

1. **Parameterization**: Make the search term ("apple") configurable
2. **Error Handling**: Add more robust error handling and notifications
3. **Incremental Loading**: Modify to only load new data rather than full refresh
4. **Data Quality Checks**: Add validation steps in the pipeline
5. **Monitoring**: Add logging and monitoring capabilities

This pipeline demonstrates a complete ELT (Extract, Load, Transform) process using modern cloud technologies, suitable for news analytics applications.


<br/>
<br/>

---

### **1. fetch_news.py**

```python
import pandas as pd
import json
import requests
import datetime
from datetime import date
import uuid
import os
from google.cloud import storage

def upload_to_gcs(bucket_name, destination_blob_name, source_file_name):
    """Uploads a file to Google Cloud Storage.
    
    Args:
        bucket_name: Name of the GCS bucket
        destination_blob_name: Path where file will be stored in GCS
        source_file_name: Local file path to upload
    """
    storage_client = storage.Client()  # Initialize GCS client
    bucket = storage_client.bucket(bucket_name)  # Get bucket reference
    blob = bucket.blob(destination_blob_name)  # Create blob object
    blob.upload_from_filename(source_file_name)  # Perform upload
    print(f"File {source_file_name} uploaded to {destination_blob_name}.")

def fetch_news_data():
    """Main function to fetch news data from NewsAPI and process it."""
    # Date setup - gets yesterday's and today's date
    today = date.today()
    api_key = '3ef02d8a5fd14f1d9615bff33b1213e3'  # NewsAPI key (should be in env vars in production)

    # API URL template with placeholders for parameters
    base_url = "https://newsapi.org/v2/everything?q={}&from={}&to={}&sortBy=popularity&apiKey={}&language=en"
    start_date_value = str(today - datetime.timedelta(days=1))  # Yesterday's date
    end_date_value = str(today)  # Today's date

    # Initialize empty DataFrame with specific columns
    df = pd.DataFrame(columns=['newsTitle', 'timestamp', 'url_source', 'content', 'source', 'author', 'urlToImage'])

    # Format API URL with parameters (query="apple")
    url_extractor = base_url.format("apple", start_date_value, end_date_value, api_key)
    response = requests.get(url_extractor)  # Make API request
    d = response.json()  # Parse JSON response

    # Process each article in the response
    for i in d['articles']:
        # Extract relevant fields
        newsTitle = i['title']
        timestamp = i['publishedAt']
        url_source = i['url']
        source = i['source']['name']
        author = i['author']
        urlToImage = i['urlToImage']
        partial_content = i['content'] if i['content'] is not None else ""  # Handle null content
        
        # Content processing - trim to 200 chars or last complete sentence
        if len(partial_content) >= 200:
            partial_content = partial_content[:199]
        if '.' in partial_content:
            trimmed_part = partial_content[:partial_content.rindex('.')]
        else:
            trimmed_part = partial_content

        # Create new row and append to DataFrame
        new_row = pd.DataFrame({
            'newsTitle': [newsTitle],
            'timestamp': [timestamp],
            'url_source': [url_source],
            'content': [trimmed_part],
            'source': [source],
            'author': [author],
            'urlToImage': [urlToImage]
        })
        df = pd.concat([df, new_row], ignore_index=True)

    # Generate timestamped filename
    current_time = datetime.datetime.now().strftime('%Y%m%d%H%M%S')
    filename = f'run_{current_time}.parquet'
    print(df)  # Debug print
    
    # Write DataFrame to Parquet file
    df.to_parquet(filename)

    # Upload to GCS
    bucket_name = 'snowflake_projects'
    destination_blob_name = f'news_data_analysis/parquet_files/{filename}'
    upload_to_gcs(bucket_name, destination_blob_name, filename)

    # Cleanup local file
    os.remove(filename)
```

---

### **2. airflow_job.py**

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
from fetch_news import fetch_news_data
from airflow.contrib.operators.snowflake_operator import SnowflakeOperator

# Default arguments for the DAG
default_args = {
    'owner': 'growdataskills',  # Owner of the DAG
    'depends_on_past': False,    # Don't depend on previous runs
    'email_on_failure': False,   # Disable failure emails
    'email_on_retry': False,     # Disable retry emails
    'retries': 0,               # Number of retries
    'retry_delay': timedelta(minutes=5),  # Delay between retries
}

# Define the DAG
dag = DAG(
    'newsapi_to_gcs',           # DAG ID
    default_args=default_args,  # Apply default args
    description='Fetch news articles and save as Parquet in GCS',  # Description
    schedule_interval=timedelta(days=1),  # Run daily
    start_date=datetime(2024, 8, 3),  # First execution date
    catchup=False,  # Don't backfill for previous periods
)

# Task 1: Fetch news data and upload to GCS
fetch_news_data_task = PythonOperator(
    task_id='newsapi_data_to_gcs',  # Unique task ID
    python_callable=fetch_news_data,  # Function to execute
    dag=dag,  # Associate with DAG
)

# Task 2: Create Snowflake table (auto-schema detection)
snowflake_create_table = SnowflakeOperator(
    task_id="snowflake_create_table",
    sql="""CREATE TABLE IF NOT EXISTS news_api_data USING TEMPLATE (
                SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
                FROM TABLE(INFER_SCHEMA (
                    LOCATION => '@news_api.PUBLIC.gcs_raw_data_stage',
                    FILE_FORMAT => 'parquet_format'
                ))
            )""",
    snowflake_conn_id="snowflake_conn"  # Airflow connection ID
)

# Task 3: Load data from GCS to Snowflake
snowflake_copy = SnowflakeOperator(
    task_id="snowflake_copy_from_stage",
    sql="""COPY INTO news_api.PUBLIC.news_api_data 
            FROM @news_api.PUBLIC.gcs_raw_data_stage
            MATCH_BY_COLUMN_NAME=CASE_INSENSITIVE 
            FILE_FORMAT = (FORMAT_NAME = 'parquet_format') 
            """,
    snowflake_conn_id="snowflake_conn"
)

# Define task dependencies
fetch_news_data_task >> snowflake_create_table >> snowflake_copy
```

---

### **3. snowflake_commands.sql**

```sql
-- Create dedicated database for news data
CREATE DATABASE news_api;
USE news_api;

-- Define Parquet file format for Snowflake to understand the files
CREATE FILE FORMAT parquet_format TYPE=parquet;

-- Create storage integration with Google Cloud Storage
-- This handles authentication between Snowflake and GCS
CREATE OR REPLACE STORAGE INTEGRATION news_data_gcs_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = GCS
ENABLED = TRUE
STORAGE_ALLOWED_LOCATIONS = ('gcs://snowflake_projects/news_data_analysis/parquet_files/');

-- View integration details to get service account info for GCP permissions
DESC INTEGRATION news_data_gcs_integration;

-- Create external stage pointing to GCS location
-- This acts as a pointer to the Parquet files in GCS
CREATE OR REPLACE STAGE gcs_raw_data_stage
URL = 'gcs://snowflake_projects/news_data_analysis/parquet_files/'
STORAGE_INTEGRATION = news_data_gcs_integration
FILE_FORMAT = (TYPE = 'PARQUET');

-- Verify stage creation
SHOW STAGES;

-- Example query to view loaded data (would work after pipeline runs)
SELECT * FROM news_api_data ORDER BY "newsTitle";
```

---

### Key Features Across All Files:

1. **fetch_news.py**:
   - Handles API communication and data transformation
   - Implements content trimming logic
   - Manages file operations and GCS uploads

2. **airflow_job.py**:
   - Orchestrates the complete workflow
   - Uses Snowflake's schema inference for easy maintenance
   - Implements proper task dependencies

3. **snowflake_commands.sql**:
   - Sets up the complete Snowflake environment
   - Configures secure GCS integration
   - Creates reusable objects for the pipeline

Each file contains detailed comments explaining the purpose of each section and important implementation details. The files work together to create a complete, automated pipeline for news data ingestion and processing.


<br/>
<br/>

# Detailed Explanation of `fetch_news.py`

This script is responsible for fetching news articles from the NewsAPI, processing the data, and uploading it to Google Cloud Storage (GCS) in Parquet format. Here's a comprehensive breakdown:

## 1. Imports and Dependencies

```python
import pandas as pd
import json
import requests
import datetime
from datetime import date
import uuid
import os
from google.cloud import storage
```

- **pandas**: For data manipulation and creating DataFrames
- **requests**: For making HTTP requests to the NewsAPI
- **datetime**: For handling dates and timestamps
- **uuid**: For generating unique identifiers (though not currently used)
- **os**: For file operations and working with paths
- **google.cloud.storage**: For interacting with Google Cloud Storage

## 2. Core Functions

### `upload_to_gcs(bucket_name, destination_blob_name, source_file_name)`

This helper function handles uploading files to Google Cloud Storage.

```python
def upload_to_gcs(bucket_name, destination_blob_name, source_file_name):
    storage_client = storage.Client()  # Creates a GCS client
    bucket = storage_client.bucket(bucket_name)  # Gets the bucket reference
    blob = bucket.blob(destination_blob_name)  # Creates a blob reference
    blob.upload_from_filename(source_file_name)  # Uploads the file
    print(f"File {source_file_name} uploaded to {destination_blob_name}.")
```

### `fetch_news_data()`

This is the main function that orchestrates the entire process:

#### A. Setup and Configuration
```python
today = date.today()
api_key = '3ef02d8a5fd14f1d9615bff33b1213e3'  # NewsAPI key
base_url = "https://newsapi.org/v2/everything?q={}&from={}&to={}&sortBy=popularity&apiKey={}&language=en"
start_date_value = str(today - datetime.timedelta(days=1))  # Yesterday's date
end_date_value = str(today)  # Today's date
```

- Uses NewsAPI's "everything" endpoint
- Searches for news about "apple" (hardcoded query)
- Gets news from yesterday to today
- Sorts by popularity and filters English language articles

#### B. Data Collection
```python
df = pd.DataFrame(columns=['newsTitle', 'timestamp', 'url_source', 'content', 'source', 'author', 'urlToImage'])
url_extractor = base_url.format("apple", start_date_value, end_date_value, api_key)
response = requests.get(url_extractor)
d = response.json()
```

- Creates an empty DataFrame with specific columns
- Formats the API URL with parameters
- Makes the HTTP request and parses JSON response

#### C. Data Processing
```python
for i in d['articles']:
    newsTitle = i['title']
    timestamp = i['publishedAt']
    url_source = i['url']
    source = i['source']['name']
    author = i['author']
    urlToImage = i['urlToImage']
    partial_content = i['content'] if i['content'] is not None else ""
    
    # Content trimming logic
    if len(partial_content) >= 200:
        partial_content = partial_content[:199]
    if '.' in partial_content:
        trimmed_part = partial_content[:partial_content.rindex('.')]
    else:
        trimmed_part = partial_content
```

- Extracts key fields from each article
- Handles null content safely
- Trims content to 200 characters or to the last complete sentence

#### D. DataFrame Construction
```python
    new_row = pd.DataFrame({
        'newsTitle': [newsTitle],
        'timestamp': [timestamp],
        'url_source': [url_source],
        'content': [trimmed_part],
        'source': [source],
        'author': [author],
        'urlToImage': [urlToImage]
    })

    df = pd.concat([df, new_row], ignore_index=True)
```

- Creates a DataFrame for each article
- Concatenates with the main DataFrame

#### E. File Output and Upload
```python
current_time = datetime.datetime.now().strftime('%Y%m%d%H%M%S')
filename = f'run_{current_time}.parquet'
print(df)

# Debugging: Print current working directory
print("Current Working Directory:", os.getcwd())

# Write to Parquet
df.to_parquet(filename)

# GCS Upload
bucket_name = 'snowflake_projects'
destination_blob_name = f'news_data_analysis/parquet_files/{filename}'
upload_to_gcs(bucket_name, destination_blob_name, filename)

# Cleanup
os.remove(filename)
```

- Generates a timestamped filename
- Saves DataFrame as Parquet file
- Uploads to GCS in the specified path
- Cleans up the local file

## Key Features and Considerations

1. **Data Structure**: Maintains consistent schema with 7 key fields
2. **Error Handling**: Basic null checks but could be enhanced
3. **Performance**: Processes articles sequentially (could parallelize)
4. **Security**: API key is hardcoded (should be in environment variables)
5. **Scalability**: Currently fetches only "apple" news (could parameterize)

## Potential Improvements

1. **Parameterization**: Make query term, date range, and other parameters configurable
2. **Error Handling**: Add retries for API calls and better error logging
3. **Memory Management**: Process articles in batches for very large result sets
4. **Metadata**: Add additional fields like processing timestamp or record count
5. **Validation**: Add data quality checks before saving

This script provides a solid foundation for a news data pipeline that could be extended for more complex use cases.


<br/>
<br/>

# Detailed Explanation of `airflow_job.py`

This Airflow DAG orchestrates a complete pipeline for fetching news data, storing it in GCS, and loading it into Snowflake. Here's a comprehensive breakdown:

## 1. Imports and Dependencies

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
from fetch_news import fetch_news_data
from airflow.contrib.operators.snowflake_operator import SnowflakeOperator
```

- **DAG**: The core Airflow class for defining workflows
- **PythonOperator**: For executing Python functions as tasks
- **datetime/timedelta**: For scheduling and time operations
- **fetch_news_data**: The custom function from our module
- **SnowflakeOperator**: For executing Snowflake SQL commands

## 2. Default Arguments

```python
default_args = {
    'owner': 'growdataskills',
    'depends_on_past': False,
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 0,
    'retry_delay': timedelta(minutes=5),
}
```

- **owner**: Identifies the DAG owner
- **depends_on_past**: Each run is independent of previous runs
- **email settings**: Disabled for simplicity (would configure in production)
- **retry settings**: No retries configured (0) but shows where to add them

## 3. DAG Definition

```python
dag = DAG(
    'newsapi_to_gcs',
    default_args=default_args,
    description='Fetch news articles and save as Parquet in GCS',
    schedule_interval=timedelta(days=1),
    start_date=datetime(2024, 8, 3),
    catchup=False,
)
```

- **name**: 'newsapi_to_gcs' - unique identifier
- **description**: Clearly states the DAG's purpose
- **schedule_interval**: Runs daily (`timedelta(days=1)`)
- **start_date**: August 3, 2024 (first execution date)
- **catchup**: Prevents backfilling past dates

## 4. Tasks Definition

### Task 1: Fetch News Data (PythonOperator)

```python
fetch_news_data_task = PythonOperator(
    task_id='newsapi_data_to_gcs',
    python_callable=fetch_news_data,
    dag=dag,
)
```

- **task_id**: Unique identifier within the DAG
- **python_callable**: The `fetch_news_data` function to execute
- **dag**: Associates this task with our DAG

### Task 2: Create Snowflake Table (SnowflakeOperator)

```python
snowflake_create_table = SnowflakeOperator(
    task_id="snowflake_create_table",
    sql="""CREATE TABLE IF NOT EXISTS news_api_data USING TEMPLATE (
                SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
                FROM TABLE(INFER_SCHEMA (
                    LOCATION => '@news_api.PUBLIC.gcs_raw_data_stage',
                    FILE_FORMAT => 'parquet_format'
                ))
            )""",
    snowflake_conn_id="snowflake_conn"
)
```

- Uses Snowflake's schema inference to automatically create a table matching the Parquet structure
- `INFER_SCHEMA` examines files in the stage to determine column names and types
- `USING TEMPLATE` creates a table with the inferred schema
- Requires a pre-configured Airflow connection named "snowflake_conn"

### Task 3: Load Data into Snowflake (SnowflakeOperator)

```python
snowflake_copy = SnowflakeOperator(
    task_id="snowflake_copy_from_stage",
    sql="""COPY INTO news_api.PUBLIC.news_api_data 
            FROM @news_api.PUBLIC.gcs_raw_data_stage
            MATCH_BY_COLUMN_NAME=CASE_INSENSITIVE 
            FILE_FORMAT = (FORMAT_NAME = 'parquet_format') 
            """,
    snowflake_conn_id="snowflake_conn"
)
```

- Copies data from GCS stage to Snowflake table
- `MATCH_BY_COLUMN_NAME=CASE_INSENSITIVE` ensures flexible column matching
- Uses the predefined 'parquet_format' file format
- Automatically handles new files in the stage

## 5. Task Dependencies

```python
fetch_news_data_task >> snowflake_create_table >> snowflake_copy
```

Defines the execution order:
1. First fetch news data and upload to GCS
2. Then ensure the Snowflake table exists
3. Finally load data from GCS into Snowflake

## Key Features

1. **Modular Design**: Separates data fetching, table creation, and data loading
2. **Snowflake Automation**: Uses schema inference for easy maintenance
3. **Incremental Loading**: Daily runs process only new data
4. **Error Handling**: Basic retry configuration available
5. **Scalability**: Can easily add more tasks or parallelize operations

## Potential Enhancements

1. **Error Handling**: Add more robust error notifications
2. **Data Validation**: Include data quality checks between tasks
3. **Monitoring**: Add sensors to verify task prerequisites
4. **Parameters**: Make query terms or date ranges configurable
5. **Backfilling**: Enable for historical data loads when needed

## Execution Flow

1. Airflow scheduler triggers the DAG daily
2. First task runs `fetch_news_data()` which:
   - Calls NewsAPI
   - Processes results
   - Uploads Parquet to GCS
3. Second task ensures the Snowflake table exists with proper schema
4. Third task loads new data from GCS into Snowflake
5. Airflow monitors task completion and reports status

This DAG provides a complete, production-ready pipeline for news data ingestion with proper separation of concerns between extraction, storage, and loading.


<br/>
<br/>

# Detailed Explanation of `snowflake_commands.sql`

This SQL file sets up the Snowflake infrastructure needed to receive and process the news data fetched by the pipeline. Here's a comprehensive breakdown:

## 1. Database Creation

```sql
CREATE DATABASE news_api;
USE news_api;
```

- Creates a dedicated database called `news_api` to isolate all news-related objects
- The `USE` statement sets this as the current database for subsequent commands

## 2. File Format Definition

```sql
CREATE FILE FORMAT parquet_format TYPE=parquet;
```

- Creates a named file format called `parquet_format`
- Specifies that files will be in Parquet format (binary columnar storage)
- This will be used when loading data from the external stage

## 3. Storage Integration Setup

```sql
CREATE OR REPLACE STORAGE INTEGRATION news_data_gcs_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = GCS
ENABLED = TRUE
STORAGE_ALLOWED_LOCATIONS = ('gcs://snowflake_projects/news_data_analysis/parquet_files/');
```

- Creates a storage integration object named `news_data_gcs_integration`
- Configures it to work with Google Cloud Storage (GCS)
- Specifies the exact bucket and path where Parquet files will be stored
- This integration handles authentication between Snowflake and GCS

## 4. Integration Verification

```sql
DESC INTEGRATION news_data_gcs_integration;
```

- Displays details about the created integration
- Shows critical information including the GCS service account that needs permissions
- Useful for verifying the integration was created correctly

## 5. External Stage Creation

```sql
CREATE OR REPLACE STAGE gcs_raw_data_stage
URL = 'gcs://snowflake_projects/news_data_analysis/parquet_files/'
STORAGE_INTEGRATION = news_data_gcs_integration
FILE_FORMAT = (TYPE = 'PARQUET');
```

- Creates an external stage named `gcs_raw_data_stage`
- Points to the GCS bucket location where Parquet files are stored
- Associates the storage integration for authentication
- Specifies that files in this stage are in Parquet format

## 6. Stage Verification

```sql
SHOW STAGES;
```

- Lists all available stages in the current database
- Verifies that `gcs_raw_data_stage` was created successfully
- Shows additional metadata about the stage

## 7. Data Query Example

```sql
SELECT * FROM news_api_data ORDER BY "newsTitle";
```

- Example query to retrieve data from the target table
- Orders results by news title (note the quotes for case sensitivity)
- This would work after the Airflow pipeline has loaded data

## Key Components and Their Roles

1. **Database**: Isolates all news-related objects
2. **File Format**: Defines how Snowflake should interpret Parquet files
3. **Storage Integration**: Secure connection between Snowflake and GCS
4. **External Stage**: Pointer to the GCS location with data files

## Execution Context

These commands would typically be run:
1. **Initially**: To set up the Snowflake environment before running the pipeline
2. **As needed**: To modify or recreate objects if requirements change
3. **Some commands**: Like the SELECT, would be run repeatedly to check data

## Security Considerations

1. The storage integration requires proper IAM permissions in GCP
2. Snowflake needs appropriate privileges to create these objects
3. The GCS bucket must allow access from Snowflake's service account

## Relationship to Airflow Pipeline

The Airflow DAG uses these objects when:
1. Creating the table (references the stage in INFER_SCHEMA)
2. Loading data (uses the stage and file format in COPY INTO)
3. The storage integration enables secure access to GCS

This setup provides a robust foundation for continuously loading news data into Snowflake with minimal manual intervention.

<br/>
<br/>


# Project Workflow: News Data Pipeline

Here's a detailed breakdown of the end-to-end flow of this project, including sample input/output at each stage:

## 1. Overall Architecture Flow

```
[NewsAPI] → [Airflow DAG] → [GCS Bucket] → [Snowflake] → [Analysis/Reporting]
```

## 2. Detailed Step-by-Step Execution Flow

### **Day 1: Initial Setup**
1. **Snowflake Configuration** (Manual one-time setup)
   - Execute `snowflake_commands.sql` in Snowflake
   - Creates database, file format, storage integration, and stage
   - Output: Snowflake objects ready to receive data

2. **Airflow Deployment**
   - Place `fetch_news.py` and `airflow_job.py` in Airflow's DAGs folder
   - Configure "snowflake_conn" connection in Airflow

### **Day 2: First Automated Run (August 3, 2024)**
**Time: 00:00 UTC**

#### **Task 1: Fetch News Data (newsapi_data_to_gcs)**
- **Input**: None (uses current date)
- **Process**:
  - Calls NewsAPI with query="apple", date_range=yesterday-to-today
  - Sample API Request:
    ```
    GET https://newsapi.org/v2/everything?q=apple&from=2024-08-02&to=2024-08-03&sortBy=popularity&apiKey=3ef02d8a5fd14f1d9615bff33b1213e3&language=en
    ```
  - Receives 20-100 articles (typical API response)
- **Transformation**:
  - Trims content to 200 chars or last full sentence
  - Creates DataFrame with 7 columns
- **Output**:
  - Parquet file: `run_20240803000000.parquet`
  - GCS Path: `gs://snowflake_projects/news_data_analysis/parquet_files/run_20240803000000.parquet`
  - Sample file content (1 row example):
    ```json
    {
      "newsTitle": "Apple announces new AI features",
      "timestamp": "2024-08-02T14:30:00Z",
      "url_source": "https://example.com/apple-ai",
      "content": "Apple unveiled new AI capabilities...",
      "source": "Tech News",
      "author": "John Doe",
      "urlToImage": "https://example.com/image1.jpg"
    }
    ```

#### **Task 2: Create Snowflake Table (snowflake_create_table)**
- **Input**: Schema inferred from GCS stage
- **Process**:
  - Runs `INFER_SCHEMA` on the Parquet file
  - Creates `news_api_data` table with matching structure
- **Output**:
  - Table schema:
    ```
    NEWSAPI.PUBLIC.NEWS_API_DATA (
      newsTitle STRING,
      timestamp TIMESTAMP_NTZ,
      url_source STRING,
      content STRING,
      source STRING,
      author STRING,
      urlToImage STRING
    )
    ```

#### **Task 3: Load Data to Snowflake (snowflake_copy_from_stage)**
- **Input**: New Parquet file in GCS
- **Process**:
  - Executes COPY INTO command
  - Loads all new files from the stage
- **Output**:
  - 20-100 rows added to `news_api_data`
  - Sample query result:
    ```sql
    SELECT newsTitle, source, timestamp 
    FROM news_api_data 
    ORDER BY timestamp DESC LIMIT 3;
    ```
    Result:
    | newsTitle                        | source      | timestamp           |
    |----------------------------------|-------------|---------------------|
    | Apple announces new AI features  | Tech News   | 2024-08-02 14:30:00 |
    | iPhone sales exceed expectations | Financial   | 2024-08-02 12:15:00 |
    | Apple store opens in Dubai       | Local News  | 2024-08-02 09:45:00 |

### **Day 3: Subsequent Run (August 4, 2024)**
- Same flow repeats with new data:
  - New file: `run_20240804000000.parquet`
  - Only new articles from 2024-08-03 to 2024-08-04 loaded
  - Table maintains all historical data

## 3. Sample Scenario with Actual Data

**Input (API Response Snippet):**
```json
{
  "articles": [
    {
      "title": "Apple Vision Pro gets major update",
      "publishedAt": "2024-08-03T08:45:00Z",
      "url": "https://tech.com/vision-pro-update",
      "source": {"name": "Tech Today"},
      "author": "Sarah Chen",
      "urlToImage": "https://tech.com/images/vision.jpg",
      "content": "Apple released visionOS 2.0 with new features... Full details at developer.apple.com."
    }
  ]
}
```

**Pipeline Processing:**
1. Trims content to: "Apple released visionOS 2.0 with new features..."
2. Creates Parquet row with all fields

**Snowflake Output:**
```sql
SELECT * FROM news_api_data WHERE timestamp > '2024-08-03';
```
Result:
| newsTitle                     | timestamp           | url_source                          | content                                      | source     | author     | urlToImage                     |
|-------------------------------|---------------------|-------------------------------------|----------------------------------------------|------------|------------|--------------------------------|
| Apple Vision Pro gets update   | 2024-08-03 08:45:00| https://tech.com/vision-pro-update  | Apple released visionOS 2.0 with new features| Tech Today | Sarah Chen | https://tech.com/images/vision.jpg |

## 4. Error Handling Flow

If NewsAPI fails:
1. PythonOperator task fails
2. Airflow marks DAG as failed
3. No new files appear in GCS
4. Subsequent tasks don't execute
5. Manual intervention or retries needed

## 5. Maintenance Aspects

1. **Storage Rotation**:
   - GCS: Old Parquet files can be archived/deleted
   - Snowflake: Time travel retains data for set period

2. **Schema Evolution**:
   - If new fields added to API response:
     - `INFER_SCHEMA` detects new columns
     - Table automatically updates (if using CREATE OR REPLACE)

This pipeline provides a complete, automated solution for daily news data ingestion with minimal maintenance requirements once deployed.

<br/>
<br/>

# Connecting Airflow to Snowflake

To establish a connection between Apache Airflow and Snowflake, you'll need to follow these steps:

## 1. Install Required Packages

First, ensure you have the necessary Python packages installed in your Airflow environment:

```bash
pip install apache-airflow-providers-snowflake
```

## 2. Create a Snowflake Connection in Airflow

### Method A: Using Airflow UI
1. Open the Airflow web interface
2. Navigate to **Admin** → **Connections**
3. Click the **+** button to add a new connection
4. Fill in the connection details:

   - **Connection Id**: `snowflake_conn` (matches what's in your DAG)
   - **Connection Type**: `Snowflake`
   - **Host**: Your Snowflake account URL (e.g., `xy12345.us-east-1.snowflakecomputing.com`)
   - **Schema**: Your default schema (e.g., `PUBLIC`)
   - **Login**: Your Snowflake username
   - **Password**: Your Snowflake password
   - **Account**: Your Snowflake account identifier (e.g., `xy12345`)
   - **Warehouse**: Your compute warehouse (e.g., `COMPUTE_WH`)
   - **Database**: Your default database (e.g., `NEWS_API`)
   - **Region**: Your Snowflake region (e.g., `us-east-1`)

### Method B: Using CLI or Code

```bash
airflow connections add snowflake_conn \
    --conn-type snowflake \
    --conn-host xy12345.us-east-1.snowflakecomputing.com \
    --conn-login your_username \
    --conn-password your_password \
    --conn-extra '{"account": "xy12345", "warehouse": "COMPUTE_WH", "database": "NEWS_API", "region": "us-east-1"}'
```

## 3. Verify the Connection

Create a test DAG to verify the connection:

```python
from airflow import DAG
from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator
from datetime import datetime

with DAG(
    'test_snowflake_connection',
    start_date=datetime(2023, 1, 1),
    schedule_interval=None,
    catchup=False
) as dag:
    
    test_query = SnowflakeOperator(
        task_id='test_query',
        sql='SELECT CURRENT_VERSION()',
        snowflake_conn_id='snowflake_conn'
    )
```

## 4. Connection Configuration Options

You can configure these additional parameters in the "Extra" field as JSON:

```json
{
    "role": "SYSADMIN",
    "authenticator": "snowflake",
    "session_parameters": {
        "QUERY_TAG": "Airflow"
    },
    "application": "AIRFLOW"
}
```

## 5. Key Security Considerations

1. **For Production**:
   - Use SSO or key pair authentication instead of passwords
   - Consider using Airflow's Secret Backend for credentials
   - Limit permissions of the Snowflake user

2. **Authentication Options**:
   - Password authentication (shown above)
   - Key pair authentication
   - OAuth (for SSO integration)

## 6. Troubleshooting Tips

If you encounter issues:
1. Verify network connectivity between Airflow and Snowflake
2. Check the Snowflake user has proper permissions
3. Examine Airflow logs for detailed error messages
4. Test the connection credentials using SnowSQL

## Best Practices

1. Use separate connections for different environments (dev/stage/prod)
2. Implement connection rotation for credentials
3. Monitor connection usage in Snowflake
4. Consider using connection pooling for high-frequency operations

This connection setup will enable your Airflow DAG to execute Snowflake operations as shown in your `airflow_job.py` file, including creating tables and loading data.


<br/>
<br/>

# **Incremental Data Flow in Snowflake**

#### **1. Automatic Incremental Loading (COPY INTO)**
- **Mechanism**:  
  The `COPY INTO` command in Snowflake is **idempotent** by default. It:
  1. Tracks which files have already been loaded via metadata
  2. Only processes **new/unloaded files** in the GCS stage
  3. Skips duplicates based on file names/ETags

- **Evidence in Your Code**:
  ```sql
  COPY INTO news_api.PUBLIC.news_api_data
  FROM @news_api.PUBLIC.gcs_raw_data_stage
  -- No explicit "incremental" flag needed - this is default behavior
  ```

#### **2. File-Based Incremental Processing**
- **How It Works**:
  - Each Airflow run generates a **new uniquely named Parquet file** (e.g., `run_20240803000000.parquet`)
  - Snowflake's stage remembers previously loaded files
  - Only the **new daily file** gets loaded in subsequent runs

- **Key Advantage**:  
  No need for complex "last updated timestamp" logic since file naming ensures uniqueness.

#### **3. Schema Evolution Handling**
- **If New Fields Appear**:
  Snowflake's `INFER_SCHEMA` in your `snowflake_create_table` task will:
  1. Detect new columns in incoming Parquet files
  2. Automatically alter the table schema (when using `USING TEMPLATE`)
  3. Preserve existing data while adding new columns

#### **4. Time-Travel for Accident Protection**
- **Built-in Safety Net**:
  - Snowflake automatically retains historical data for:
    - Standard edition: 1 day
    - Enterprise: Up to 90 days (configurable)
  - Allows querying previous states:
    ```sql
    SELECT * FROM news_api_data AT(OFFSET => -60*60); -- Data 1 hour ago
    ```

#### **5. Sample Incremental Execution**

**Day 1 (2024-08-03)**:
- Loads 50 articles from `run_20240803000000.parquet`
- Table count: 50 rows

**Day 2 (2024-08-04)**:
- New file: `run_20240804000000.parquet` with 30 articles
- COPY INTO:
  - Detects only the new file needs loading
  - Adds 30 rows
- Table count: 80 rows (50 + 30)

**Verification Query**:
```sql
-- See data partitioning by load time
SELECT 
  DATE(timestamp) as news_date,
  COUNT(*) as articles,
  MIN(_load_time) as first_load_at
FROM news_api_data
GROUP BY 1 ORDER BY 1;
```

#### **6. Performance Optimization**
- **Micro-Partitioning**:  
  Snowflake automatically partitions data by:
  - `timestamp` column (from the news data)
  - `_load_time` (when the row was ingested)
- **Clustering**:  
  For large datasets, you could add:
  ```sql
  ALTER TABLE news_api_data CLUSTER BY (DATE(timestamp));
  ```

#### **7. Handling Updates (If Needed)**
For scenarios where news articles might be updated:
1. **Option 1**: Use `MERGE` statement
   ```sql
   MERGE INTO news_api_data t
   USING (SELECT * FROM @stage/new_file.parquet) s
   ON t.url_source = s.url_source -- Use URL as natural key
   WHEN MATCHED THEN UPDATE SET ...
   WHEN NOT MATCHED THEN INSERT ...;
   ```
2. **Option 2**: Append-only with versioning
   ```sql
   -- Add version column during load
   COPY INTO news_api_data 
   SELECT *, CURRENT_TIMESTAMP() as _version FROM @stage;
   ```

#### **Key Advantages of This Approach**
1. **No Custom Logic Needed**: Leverages Snowflake's built-in incremental capabilities
2. **Fault Tolerant**: Failed loads can be retried safely
3. **Transparent**: View load history with:
   ```sql
   SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
     table_name => 'NEWS_API_DATA', 
     start_time => DATEADD(hour, -24, CURRENT_TIMESTAMP())));
   ```

This design ensures efficient daily loading without duplications while maintaining full historical data - all with minimal custom code.

<br/>
<br/>

# **FAQs Based on This News Data Pipeline Project**

#### **1. Snowflake Setup & Configuration**
**Q1: Why do we need to create a Snowflake stage before running the Airflow DAG?**  
**A:** The stage (`gcs_raw_data_stage`) acts as a pointer to the GCS bucket where Parquet files are stored. Airflow’s `COPY INTO` command references this stage to load data. Without it, Snowflake wouldn’t know where to fetch the files.

**Q2: How does Snowflake ensure no duplicate data is loaded?**  
**A:** Snowflake’s `COPY INTO` command is idempotent. It tracks loaded files via metadata (e.g., file names/ETags) and skips duplicates automatically.

**Q3: What happens if the schema of incoming Parquet files changes?**  
**A:** The `INFER_SCHEMA` in the `snowflake_create_table` task detects new columns and auto-updates the table schema (if using `USING TEMPLATE`). Existing data remains intact.

---

#### **2. Airflow & Orchestration**
**Q4: Why use Airflow instead of running the Python script directly?**  
**A:** Airflow provides:  
- **Scheduling** (daily runs)  
- **Dependency management** (ensure Snowflake setup completes first)  
- **Retries/Failure handling** (auto-retry failed tasks)  
- **Monitoring** (track pipeline history via UI).

**Q5: How would you handle API failures in the `fetch_news_data` task?**  
**A:** Options:  
- Add `retries=3` in the `PythonOperator`.  
- Use a `try-catch` block in the script to log errors.  
- Implement Airflow’s `on_failure_callback` to alert via Slack/email.

**Q6: Why is the Snowflake connection configured in Airflow and not hardcoded?**  
**A:** Hardcoding credentials is insecure. Airflow’s connection store:  
- Encrypts credentials.  
- Allows reuse across DAGs.  
- Simplifies rotation (update once, not in every script).

---

#### **3. Data Processing & GCS**
**Q7: Why trim article content to 200 characters?**  
**A:** To:  
- Reduce storage costs.  
- Avoid loading huge text blobs into Snowflake.  
- Improve query performance (smaller data → faster scans).

**Q8: How would you backfill historical news data?**  
**A:** Options:  
- Temporarily set `catchup=True` in the DAG and adjust `start_date`.  
- Manually run `fetch_news_data` with a custom date range.  
- Use Snowflake’s `COPY INTO` with `FORCE=TRUE` to reload files.

**Q9: Why use Parquet instead of CSV/JSON?**  
**A:** Parquet offers:  
- Columnar storage (better compression/query performance).  
- Schema evolution support.  
- Native integration with Snowflake.

---

#### **4. Incremental Loading & Performance**
**Q10: How does Snowflake know which files are new each day?**  
**A:** The pipeline:  
1. Generates unique filenames (e.g., `run_<timestamp>.parquet`).  
2. Snowflake’s `COPY INTO` skips files already processed (tracked internally).

**Q11: What if a news article is updated after being loaded?**  
**A:** Options:  
- Use `MERGE` statements to update existing records (via `url_source` as a key).  
- Treat updates as new rows and add versioning (e.g., `valid_from/valid_to` timestamps).  

**Q12: How would you optimize query performance in Snowflake?**  
**A:**  
- Cluster by `timestamp`: `ALTER TABLE news_api_data CLUSTER BY (DATE(timestamp));`.  
- Use Snowflake’s automatic clustering (if Enterprise edition).  
- Materialize aggregations (e.g., daily article counts).

---

#### **5. Security & Maintenance**
**Q13: How is the NewsAPI key secured?**  
**A:** It should be:  
- Stored in Airflow’s **Variables** (not hardcoded).  
- Encrypted via Airflow’s secrets backend (e.g., HashiCorp Vault).  
- Restricted to specific IPs (if supported by NewsAPI).

**Q14: How would you monitor pipeline health?**  
**A:**  
- Airflow: Check DAG run history/alerts.  
- Snowflake: Query `COPY_HISTORY()` for load metrics.  
- GCS: Monitor bucket storage growth.  

**Q15: What’s the cost impact of this pipeline?**  
**A:** Factors:  
- NewsAPI: Free tier limits (500 requests/day).  
- GCS: ~$0.02/GB/month storage.  
- Snowflake: Storage + compute (warehouse usage during loads).  

---

#### **6. Extensions & Scaling**
**Q16: How would you add sentiment analysis to news articles?**  
**A:** Options:  
- Use Snowflake’s **Snowpark ML** to run Python sentiment models.  
- Call an external API (e.g., AWS Comprehend) in `fetch_news.py`.  

**Q17: What if you need to fetch news for multiple keywords (not just "apple")?**  
**A:**  
- Parameterize the query term in `fetch_news.py` (e.g., via Airflow `params`).  
- Use dynamic task generation in Airflow to parallelize fetches.  

**Q18: How would you share this data with analysts?**  
**A:**  
- Create Snowflake views: `CREATE VIEW tech_news AS SELECT * FROM news_api_data WHERE content LIKE '%apple%';`.  
- Set up row-level security (RLS) if needed.  

---

### **Key Themes in FAQs**
1. **Idempotency**: How Snowflake avoids duplicates.  
2. **Schema Evolution**: Handling new fields.  
3. **Error Recovery**: Retries/backfills.  
4. **Cost vs. Performance**: Tradeoffs in design.  
5. **Extensibility**: Adding new features.  

These questions cover architectural, operational, and optimization aspects of the project.