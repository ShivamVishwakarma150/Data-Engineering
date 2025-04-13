# **Capital Market Data Analytics Project: Real-Time Risk Monitoring System**

## **Project Title:**  
**Incremental Risk Analytics for Capital Markets using Databricks**  

## **Objective:**  
Build a **real-time risk monitoring system** that processes incremental market data (stock prices, trades, orders) and computes key risk metrics (VaR, volatility, liquidity risk) using **Databricks Delta Lake and Structured Streaming**.

---

## **Key Features & Components**  

### **1. Data Sources (Incremental Inputs)**
- **Stock Market Feeds** (e.g., NYSE/NASDAQ real-time tick data)
- **Order Book Updates** (bid/ask changes, trade executions)
- **Corporate Actions** (splits, dividends, M&A announcements)
- **Economic Indicators** (interest rates, inflation data)

### **2. Data Pipeline Architecture (Using Databricks)**
```
Raw Market Data (Bronze) → Cleaned & Enriched Data (Silver) → Risk Metrics (Gold)
```
- **Bronze Layer (Raw Ingestion)**  
  - Ingest real-time market data via **Kafka, AWS Kinesis, or Delta Live Tables (DLT)**  
  - Store as **Delta tables** for ACID compliance  

- **Silver Layer (Processing & Validation)**  
  - **Clean & normalize** tick data (handle missing values, outliers)  
  - **Join with reference data** (company info, sector classifications)  
  - **Apply SCD Type 2** for corporate actions tracking  

- **Gold Layer (Risk Analytics)**  
  - **Compute real-time risk metrics**:  
    - **Value-at-Risk (VaR)** – Monte Carlo simulations  
    - **Liquidity Risk** – Order book depth analysis  
    - **Volatility Clustering** – GARCH modeling  
  - **Anomaly detection** (sudden price movements, flash crashes)  

### **3. Incremental Processing with Databricks**
- **Structured Streaming** for real-time risk calculations  
- **Delta Lake’s MERGE INTO** for SCD (Type 2) handling  
- **Optimized Auto Loader** for efficient file ingestion  

### **4. Output & Alerts**
- **Dashboard** (Plotly/Databricks SQL Dashboards)  
  - Real-time risk exposure heatmaps  
  - Portfolio stress testing scenarios  
- **Automated Alerts** (Slack/Email on anomalies)  

---

## **Why Databricks?**
✅ **Delta Lake** – Efficient incremental updates  
✅ **Structured Streaming** – Low-latency risk monitoring  
✅ **ML Integration** – Predictive risk models (e.g., LSTM for volatility forecasting)  
✅ **Scalability** – Handles petabytes of market data  

---

## **Expected Outcome**
A **scalable, real-time risk analytics platform** that helps traders and compliance teams monitor market risks efficiently, reducing exposure to sudden market crashes.  


<br/>
<br/>

# **Step-by-Step Implementation: Real-Time Risk Monitoring System for Capital Markets using Databricks**

## **Phase 1: Setup & Data Ingestion (Bronze Layer)**
### **Step 1: Databricks Environment Setup**
1. **Create a Databricks Workspace**  
   - Use **Azure Databricks/AWS Databricks**  
   - Set up **cluster** with **Delta Lake, Spark Streaming, and ML runtime**  

2. **Install Required Libraries**  
   ```python
   %pip install yfinance pandas-ta plotly pyspark pandas numpy scipy riskfolio-lib
   ```

3. **Set Up Delta Lake Tables**  
   ```python
   spark.sql("CREATE DATABASE IF NOT EXISTS capital_market")
   ```

---

### **Step 2: Ingest Real-Time Market Data**
#### **Option A: Simulated Data (For Testing)**
```python
import yfinance as yf
from pyspark.sql.functions import current_timestamp

# Fetch sample stock data (e.g., AAPL, MSFT)
stocks = ["AAPL", "MSFT", "GOOGL"]
data = yf.download(stocks, period="1d", interval="1m")

# Convert to Spark DataFrame and write to Delta (Bronze)
df = spark.createDataFrame(data.reset_index())
df = df.withColumn("ingest_time", current_timestamp())
df.write.format("delta").mode("append").saveAsTable("capital_market.bronze_stock_ticks")
```

#### **Option B: Real-Time Streaming (Production)**
```python
from pyspark.sql.streaming import DataStreamReader

# Read from Kafka (NYSE/NASDAQ feed)
kafka_stream = (spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "kafka_server:9092")
  .option("subscribe", "stock-ticks")
  .load())

# Parse JSON and store in Delta Lake
stream_query = (kafka_stream
  .selectExpr("CAST(value AS STRING) as json")
  .writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/delta/checkpoints/bronze_stocks")
  .start("/delta/bronze_stock_ticks"))
```

---

## **Phase 2: Data Processing (Silver Layer)**
### **Step 3: Clean & Normalize Data**
```python
from pyspark.sql.functions import col, when

silver_stocks = (spark.readStream
  .format("delta")
  .load("/delta/bronze_stock_ticks")
  .filter(col("price").isNotNull())  # Remove bad ticks
  .withColumn("adjusted_price", 
      when(col("volume") > 1000, col("price")).otherwise(None))  # Filter illiquid ticks
  .withWatermark("event_time", "5 minutes")  # Late data handling
)

# Write to Silver Delta Table
(silver_stocks.writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/delta/checkpoints/silver_stocks")
  .start("/delta/silver_stock_ticks"))
```

### **Step 4: SCD Type 2 for Corporate Actions**
```sql
-- Create SCD Type 2 dimension for stocks
CREATE TABLE capital_market.dim_stock_scd2 (
  stock_sk BIGINT GENERATED ALWAYS AS IDENTITY,
  stock_id STRING,
  company_name STRING,
  sector STRING,
  start_date TIMESTAMP,
  end_date TIMESTAMP,
  is_current BOOLEAN
) USING DELTA;

-- MERGE logic for updates (e.g., stock split)
MERGE INTO capital_market.dim_stock_scd2 AS target
USING (
  SELECT 
    stock_id, 
    company_name, 
    sector,
    event_time as start_date
  FROM capital_market.silver_stock_events
) AS source
ON target.stock_id = source.stock_id AND target.is_current = true
WHEN MATCHED AND target.sector <> source.sector THEN
  UPDATE SET 
    end_date = source.start_date,
    is_current = false
WHEN NOT MATCHED THEN
  INSERT (stock_id, company_name, sector, start_date, end_date, is_current)
  VALUES (source.stock_id, source.company_name, source.sector, 
          source.start_date, NULL, true);
```

---

## **Phase 3: Risk Analytics (Gold Layer)**
### **Step 5: Compute Real-Time VaR (Value-at-Risk)**
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import pandas_udf
import numpy as np
import riskfolio as rp

# Define VaR calculation (Historical Simulation)
@pandas_udf("double")
def calculate_var(prices: pd.Series) -> float:
    returns = prices.pct_change().dropna()
    var_95 = np.percentile(returns, 5)
    return abs(var_95)  # Return positive VaR

# Apply rolling VaR (1-hour window)
window_spec = Window.partitionBy("stock_id").orderBy("event_time").rowsBetween(-60, 0)
risk_metrics = (spark.table("capital_market.silver_stock_ticks")
  .withColumn("var_95", calculate_var(col("price")).over(window_spec)))
```

### **Step 6: Liquidity Risk (Order Book Imbalance)**
```python
order_book_imbalance = (spark.table("capital_market.silver_order_book")
  .withColumn("mid_price", (col("best_bid") + col("best_ask")) / 2)
  .withColumn("imbalance", 
      (col("bid_volume") - col("ask_volume")) / (col("bid_volume") + col("ask_volume")))
  .withColumn("liquidity_risk", when(col("imbalance") > 0.7, "HIGH").otherwise("LOW"))
```

---

## **Phase 4: Visualization & Alerts**
### **Step 7: Databricks Dashboard**
1. **Create a Dashboard**  
   - Use **Databricks SQL Dashboard**  
   - Panels:  
     - Real-time **VaR heatmap** by sector  
     - **Liquidity risk alerts**  
     - **Volatility trends**  

2. **Plotly for Advanced Viz**  
```python
import plotly.express as px
fig = px.line(risk_metrics.toPandas(), x="event_time", y="var_95", color="stock_id")
fig.show()
```

### **Step 8: Alerting System**
```python
# Send Slack alert if VaR exceeds threshold
high_risk_stocks = risk_metrics.filter(col("var_95") > 0.05)

if high_risk_stocks.count() > 0:
    import requests
    webhook_url = "slack_webhook_url"
    message = {"text": f"🚨 High VaR Alert: {high_risk_stocks.select('stock_id').collect()}"}
    requests.post(webhook_url, json=message)
```

---

## **Phase 5: Deployment & Scaling**
### **Step 9: Production Deployment**
1. **Schedule Jobs**  
   - Use **Databricks Workflows** for daily batch risk reports  
   - **Structured Streaming** for real-time processing  

2. **Optimize Performance**  
   ```python
   spark.conf.set("spark.databricks.delta.optimizeWrite.enabled", "true")
   spark.conf.set("spark.databricks.delta.autoCompact.enabled", "true")
   ```

3. **Monitor Pipeline Health**  
   - **Databricks Lakeview** for data quality checks  
   - **Alert on failed streams**  

---

## **Final Output**
✅ **Real-time risk dashboard**  
✅ **Automated alerts** for VaR breaches  
✅ **Historical trend analysis**  
✅ **Scalable to 1000s of stocks**  

<br/>
<br/>

# **Inputs and Outputs of the Real-Time Risk Monitoring System**

## **Input Data Sources**

### **1. Market Data Feeds (Primary Inputs)**
| Data Type | Example Sources | Frequency | Volume Estimate |
|-----------|----------------|-----------|-----------------|
| Real-time stock ticks | NYSE, NASDAQ, Bloomberg API | Milliseconds | ~10GB/day for 500 stocks |
| Order book updates | L2 market data (bid/ask spreads) | Seconds | ~5GB/day |
| Corporate actions | SEC filings, Refinitiv | Daily | <1GB/day |
| Economic indicators | FRED, World Bank | Weekly/Monthly | Minimal |

### **2. Reference Data**
| Data Type | Purpose | Example |
|-----------|---------|---------|
| Stock master | Company metadata | `{symbol: AAPL, name: Apple Inc., sector: Tech}` |
| Holiday calendar | Market closures | `2024-01-01: New Year's Day` |
| Risk parameters | VaR confidence levels | `var_confidence: 95%` |

## **Output Deliverables**

### **1. Real-Time Risk Metrics (Primary Outputs)**
| Metric | Calculation Method | Update Frequency | Sample Output |
|--------|--------------------|------------------|---------------|
| Value-at-Risk (VaR) | Historical simulation (95% CI) | 5-minute rolling | `AAPL: 2.3% 1-day VaR` |
| Liquidity risk score | Order book imbalance ratio | Tick-by-tick | `MSFT liquidity: HIGH (imbalance=0.82)` |
| Volatility | GARCH(1,1) model | Hourly | `GOOGL 30-day vol: 28%` |

### **2. Alerting System Outputs**
| Alert Type | Trigger Condition | Notification Channel | Example |
|------------|-------------------|----------------------|---------|
| VaR breach | Portfolio VaR > 5% | Slack/Email | `ALERT: Tech sector VaR breached at 5.7%` |
| Flash crash | Price drop > 3% in 1min | SMS + Dashboard | `WARNING: TSLA dropped 3.2% at 14:30` |
| Liquidity crunch | Order book imbalance > 0.7 | PagerDuty | `CRITICAL: SPY liquidity risk - HIGH` |

### **3. Analytical Outputs**
| Output Type | Format | Update Frequency | Contents |
|-------------|--------|------------------|----------|
| Risk dashboard | Databricks SQL + Plotly | Real-time | Interactive heatmaps, trend charts |
| PDF report | LaTeX-generated | EOD | VaR backtesting results, stress scenarios |
| API endpoint | REST/WebSocket | On-demand | `GET /api/risk/AAPL` returns JSON risk metrics |

### **4. Data Storage Outputs**
| Layer | Storage Format | Retention Policy | Example Tables |
|-------|---------------|------------------|----------------|
| Bronze | Delta Lake | 30 days raw | `bronze_stock_ticks`, `bronze_order_book` |
| Silver | Delta Lake (Z-ordered) | 1 year cleaned | `silver_stock_ticks`, `dim_stock_scd2` |
| Gold | Delta Lake (OPTIMIZED) | 5 years aggregated | `gold_var_metrics`, `gold_liquidity_scores` |

## **Data Flow Diagram**

```
[External Feeds]  
       ↓  
[Kafka/API Ingestion] → Bronze Layer (Raw Delta)  
       ↓  
[Spark Streaming] → Silver Layer (Cleaned + SCD2)  
       ↓  
[Risk Engine] → Gold Layer (VaR/Liquidity Metrics)  
       ↓  
[Dashboard] ← [Alerts] ← [API]
```

## **Key Statistics**
- **Input Volume**: ~15-20GB/day for mid-size portfolio
- **Processing Latency**: <30 seconds for risk metrics
- **Output Data**: ~5GB/day after aggregation
- **Alert Thresholds**: Configurable via `risk_config.json`

<br/>
<br/>

Here are concrete examples of input data and corresponding output for the Capital Market Risk Monitoring System:

### Sample Inputs:

1. **Real-Time Stock Ticks (Bronze Layer Raw Input)**
```json
{
  "symbol": "AAPL",
  "timestamp": "2024-03-15T14:30:45.123Z",
  "price": 182.34,
  "volume": 2500,
  "exchange": "NASDAQ"
}
```

2. **Order Book Update (L2 Market Data)**
```csv
symbol,timestamp,bid_price,bid_size,ask_price,ask_size
MSFT,2024-03-15T14:30:47.456Z,415.12,500,415.45,300
```

3. **Corporate Action (Reference Data)**
```sql
INSERT INTO corporate_actions VALUES 
('AAPL', '2024-05-01', 'SPLIT', '4:1', true);
```

### Sample Outputs:

1. **VaR Calculation (Gold Layer Output)**
```python
{
  "portfolio_id": "TECH_GROWTH",
  "calculation_time": "2024-03-15T15:00:00Z",
  "1_day_var_95": 4.27,  # percentage
  "constituents": [
    {"symbol": "AAPL", "var_contribution": 1.82},
    {"symbol": "MSFT", "var_contribution": 1.45}
  ]
}
```

2. **Liquidity Risk Alert (Slack Message)**
```
🚨 LIQUIDITY WARNING 🚨
Symbol: TSLA
Time: 2024-03-15 15:15:30
Order Book Imbalance: 0.78 (DANGER ZONE)
Recommended Action: Reduce position
```

3. **Dashboard Visualization Data**
```json
{
  "x_axis": ["09:30", "10:00", "10:30", "11:00"],
  "series": [
    {
      "name": "Portfolio VaR",
      "data": [2.1, 2.3, 3.8, 4.2],
      "threshold": 5.0
    }
  ]
}
```

4. **SCD2 Dimension Table Output**
```sql
SELECT * FROM dim_stock_scd2 WHERE symbol = 'AAPL';
```

stock_sk | symbol | company_name  | sector  | start_date          | end_date            | is_current
---------|--------|---------------|---------|---------------------|---------------------|-----------
1001     | AAPL   | Apple Inc.    | Tech    | 2020-01-01 00:00:00 | 2024-05-01 00:00:00 | false
1002     | AAPL   | Apple Inc.    | Tech    | 2024-05-01 00:00:00 | NULL                | true

### Transformation Example:
**Input** (Raw Trade):
```json
{"symbol":"GOOGL","price":152.33,"timestamp":"2024-03-15T14:31:22Z"}
```

**After Silver Processing**:
```python
{
  "symbol": "GOOGL",
  "price": 152.33,
  "timestamp": "2024-03-15T14:31:22Z",
  "normalized_price": 152.33,
  "data_quality": "VALID",
  "ingest_time": "2024-03-15T14:31:25Z"
}
```

**Final Gold Output** (Risk Metric):
```python
{
  "symbol": "GOOGL",
  "time_window": "14:30-14:35",
  "volatility": 0.018,
  "liquidity_score": 0.67,
  "var_95": 2.41
}
```

These examples show the complete flow from raw market data inputs to processed risk intelligence outputs. Would you like me to provide sample code for any specific transformation step?