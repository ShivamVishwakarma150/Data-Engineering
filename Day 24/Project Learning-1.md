# **Uber Data Analytics Platform: Real-Time Project Explanation**

## **1. Project Overview**
The **Uber Data Analytics Platform** is designed to process and analyze ride-hailing data in real-time and batch modes. This system helps Uber optimize operations, improve customer experience, and make data-driven decisions by tracking key business metrics such as ride duration, revenue, peak demand times, cancellation rates, and rider loyalty.

---

## **2. Business Objectives**
The platform supports the following key business insights:
1. **Operational Efficiency**  
   - Average ride duration and distance per city  
   - Time taken from ride acceptance to pickup  
   - Peak demand hours in different locations  

2. **Revenue & Performance Tracking**  
   - Revenue generated per city  
   - Popular routes & destinations  
   - Driver & rider ratings  

3. **Customer & Driver Behavior**  
   - Ride cancellation rates (by riders vs. drivers)  
   - Impact of weather on ride demand  
   - Rider loyalty metrics (frequency, spend, retention)  

---

## **3. Data Model Architecture**
The system follows a **star schema** design, where a central **fact table (`Rides`)** connects to multiple **dimension tables** (`Drivers`, `Riders`, `Locations`, `Cities`, `Dates`, `Weather`).  

### **A. Dimension Tables (Descriptive Data)**
| Table | Purpose | Key Fields |
|--------|---------|------------|
| **Drivers** | Stores driver details | `driver_id`, `driver_name`, `vehicle_type`, `signup_date` |
| **Riders** | Stores rider details | `rider_id`, `rider_name`, `loyalty_status`, `signup_date` |
| **Locations** | Tracks pickup/drop-off points | `location_id`, `city_id`, `latitude`, `longitude` |
| **Cities** | Maps locations to cities | `city_id`, `city_name`, `state`, `country` |
| **Dates** | Time-based analysis | `date_time`, `day`, `month`, `year`, `hour` |
| **Weather** | Weather conditions during rides | `weather_id`, `city_id`, `weather_condition`, `temperature` |

### **B. Fact Tables (Transactional Data)**
| Table | Purpose | Key Fields |
|--------|---------|------------|
| **Rides** | Core ride data | `ride_id`, `driver_id`, `rider_id`, `fare`, `ride_duration`, `rating_by_driver/rider` |
| **RideStatus** | Tracks ride lifecycle (real-time updates) | `status_id`, `ride_id`, `status` (`accepted`, `started`, `completed`, `cancelled`), `status_time` |

---

## **4. Real-Time Data Flow**
1. **Data Ingestion**  
   - Uber’s mobile app & driver app generate ride events (request, acceptance, pickup, drop-off, cancellation).  
   - Real-time streaming via **Kafka/Apache Pulsar**.  
   - Batch data (weather, city info) loaded via **ETL pipelines (Airflow, Spark)**.  

2. **Stream Processing**  
   - **Apache Flink/Spark Streaming** processes ride events in real-time.  
   - Updates `RideStatus` for live tracking (e.g., "Driver accepted ride in 2 mins").  
   - Aggregates metrics (e.g., "Peak demand in NYC at 5 PM").  

3. **Storage & Analytics**  
   - **OLTP Database (PostgreSQL/MySQL)** for transactional queries (e.g., "Find all rides by a driver").  
   - **OLAP Database (Snowflake/BigQuery)** for analytics (e.g., "Revenue per city last month").  
   - **Redis/Cache** for low-latency lookups (e.g., "Current ride status").  

4. **Dashboarding & Alerts**  
   - **Tableau/Power BI** dashboards for business insights.  
   - **Alerts** for anomalies (e.g., "Cancellation rate spiked in Chicago").  

---

## **5. Key SQL Queries (Business Insights)**
### **A. Operational Metrics**
```sql
-- 1. Average ride duration & distance per city
SELECT c.city_name, AVG(r.ride_duration), AVG(r.ride_distance)
FROM Rides r JOIN Cities c ON r.city_id = c.city_id
GROUP BY c.city_name;

-- 2. Time from acceptance to pickup (driver efficiency)
SELECT d.driver_id, AVG(TIMESTAMPDIFF(MINUTE, rs1.status_time, rs2.status_time))
FROM RideStatus rs1
JOIN RideStatus rs2 ON rs1.ride_id = rs2.ride_id
JOIN Rides r ON rs1.ride_id = r.ride_id
JOIN Drivers d ON r.driver_id = d.driver_id
WHERE rs1.status = 'accepted' AND rs2.status = 'started'
GROUP BY d.driver_id;
```

### **B. Revenue & Demand Analysis**
```sql
-- 3. Revenue per city (completed rides only)
SELECT c.city_name, SUM(r.fare)
FROM Rides r JOIN Cities c ON r.city_id = c.city_id
WHERE EXISTS (SELECT 1 FROM RideStatus rs WHERE rs.ride_id = r.ride_id AND rs.status = 'completed')
GROUP BY c.city_name;

-- 4. Peak demand hours in NYC
SELECT d.hour, COUNT(*)
FROM Rides r JOIN Dates d ON r.ride_date_time = d.date_time
WHERE r.city_id = 'NYC'
GROUP BY d.hour ORDER BY COUNT(*) DESC;
```

### **C. Cancellation & Weather Impact**
```sql
-- 5. Cancellation rates by riders vs. drivers
SELECT 
   (SELECT COUNT(*) FROM RideStatus WHERE status = 'cancelled_by_rider') * 100.0 / 
   (SELECT COUNT(*) FROM Rides) AS rider_cancel_rate,
   (SELECT COUNT(*) FROM RideStatus WHERE status = 'cancelled_by_driver') * 100.0 / 
   (SELECT COUNT(*) FROM Rides) AS driver_cancel_rate;

-- 6. Rides affected by rain
SELECT w.weather_condition, COUNT(*) 
FROM Rides r JOIN Weather w ON r.city_id = w.city_id 
WHERE w.weather_condition = 'rain' 
GROUP BY w.weather_condition;
```

### **D. Rider Loyalty & Retention**
```sql
-- 7. Top riders by frequency & spend
SELECT ri.rider_name, COUNT(*) as rides, SUM(r.fare) as total_spend
FROM Rides r JOIN Riders ri ON r.rider_id = ri.rider_id
GROUP BY ri.rider_name ORDER BY total_spend DESC;

-- 8. Inactive riders (no rides in 30 days)
SELECT ri.rider_name, MAX(r.ride_date_time) as last_ride_date
FROM Riders ri LEFT JOIN Rides r ON ri.rider_id = r.rider_id
GROUP BY ri.rider_name
HAVING DATEDIFF(NOW(), last_ride_date) > 30;
```

---

## **6. Tech Stack**
| Component | Technology |
|-----------|------------|
| **Data Ingestion** | Kafka, AWS Kinesis |
| **Stream Processing** | Apache Flink, Spark Streaming |
| **Batch Processing** | Apache Spark, Airflow |
| **Databases** | PostgreSQL (OLTP), Snowflake (OLAP), Redis (Cache) |
| **Dashboarding** | Tableau, Power BI, Grafana |
| **Infrastructure** | AWS/GCP, Kubernetes (for scaling) |

---

## **7. Business Impact**
✅ **Optimized Pricing** – Identify peak hours for surge pricing.  
✅ **Improved Driver Allocation** – Reduce wait times in high-demand zones.  
✅ **Reduced Cancellations** – Detect patterns (e.g., drivers cancelling long rides).  
✅ **Enhanced Loyalty Programs** – Reward frequent riders.  
✅ **Weather-Based Strategies** – Increase incentives during bad weather.  

---

## **8. Future Enhancements**
- **Predictive Analytics** (ML models for demand forecasting).  
- **Dynamic Pricing AI** (real-time fare adjustments).  
- **Fraud Detection** (identify fake ride requests).  

---

### **Conclusion**
This **real-time Uber analytics platform** provides actionable insights into ride operations, revenue, and customer behavior. By leveraging **streaming data + batch processing**, Uber can make data-driven decisions to improve efficiency, reduce cancellations, and enhance rider/driver experiences.  

<br/>
<br/>

# Uber Data Model Design

## Business Metrics Supported
The data model is designed to support the following key business metrics:
1. Average ride duration and distance per city
2. Revenue generated per city
3. Peak times for rides in different locations
4. Popular routes and destinations
5. Average rider rating per driver
6. Average driver rating per rider
7. Time from ride acceptance to customer pickup
8. Ride cancellation rates by riders and drivers
9. Impact of weather on ride demand
10. Rider loyalty metrics (frequency, spend, relationship length)

## Dimension Tables

### Drivers
```
driver_id (PK)
driver_name
driver_phone_number
driver_email
signup_date
vehicle_type
driver_city_id (FK)
```

### Riders
```
rider_id (PK)
rider_name
rider_phone_number
rider_email
signup_date
loyalty_status
rider_city_id (FK)
```

### Locations
```
location_id (PK)
city_id (FK)
location_name
latitude
longitude
```

### Cities
```
city_id (PK)
city_name
state
country
```

### Dates
```
date_time (PK)
day
month
year
weekday
hour
```

### Weather
```
weather_id (PK)
city_id (FK)
date_time (FK)
weather_condition (rain, sunny, snowy)
temperature
```

## Fact Tables

### Rides (Main Fact Table)
```
ride_id (PK)
driver_id (FK)
rider_id (FK)
start_location_id (FK)
end_location_id (FK)
ride_date_time (FK)
ride_duration
ride_distance
fare
rating_by_driver
rating_by_rider
```

### RideStatus (Transaction Fact Table)
```
status_id (PK)
ride_id (degenerate dimension)
status (accepted, started, completed, cancelled_by_driver, cancelled_by_rider)
status_time
```

## SQL Queries for Business Insights

1. **Average ride duration and distance per city**
```sql
SELECT c.city_name, AVG(r.ride_duration), AVG(r.ride_distance)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Cities c ON l.city_id = c.city_id
GROUP BY c.city_name;
```

2. **Revenue generated per city**
```sql
SELECT c.city_name, SUM(r.fare)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Cities c ON l.city_id = c.city_id
WHERE EXISTS (
    SELECT 1 FROM RideStatus rs 
    WHERE rs.ride_id = r.ride_id AND rs.status = 'completed'
)
GROUP BY c.city_name;
```

3. **Peak times for rides in different locations**
```sql
SELECT l.location_name, d.hour, COUNT(*)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Dates d ON r.ride_date_time = d.date_time
GROUP BY l.location_name, d.hour
ORDER BY COUNT(*) DESC;
```

4. **Popular routes and destinations**
```sql
SELECT l1.location_name AS start_location, l2.location_name AS end_location, COUNT(*) AS number_of_rides
FROM Rides r
JOIN Locations l1 ON r.start_location_id = l1.location_id
JOIN Locations l2 ON r.end_location_id = l2.location_id
GROUP BY l1.location_name, l2.location_name
ORDER BY number_of_rides DESC;
```

5. **Average rider rating per driver**
```sql
SELECT d.driver_name, AVG(r.rating_by_driver)
FROM Rides r
JOIN Drivers d ON r.driver_id = d.driver_id
WHERE r.rating_by_driver IS NOT NULL
GROUP BY d.driver_name;
```

6. **Average driver rating per rider**
```sql
SELECT ri.rider_name, AVG(r.rating_by_rider)
FROM Rides r
JOIN Riders ri ON r.rider_id = ri.rider_id
WHERE r.rating_by_rider IS NOT NULL
GROUP BY ri.rider_name;
```

7. **Time taken by a driver from ride acceptance to customer pickup**
```sql
SELECT d.driver_id, d.driver_name, r.ride_id,
TIMESTAMPDIFF(MINUTE, acceptance_status.status_time, pickup_status.status_time) AS minutes_to_pickup
FROM RideStatus acceptance_status
JOIN RideStatus pickup_status ON acceptance_status.ride_id = pickup_status.ride_id
JOIN Rides r ON r.ride_id = acceptance_status.ride_id
JOIN Drivers d ON r.driver_id = d.driver_id
WHERE acceptance_status.status = 'accepted'
AND pickup_status.status = 'started';
```

8. **Rate of ride cancellations by riders and drivers**
```sql
SELECT
(SELECT COUNT(DISTINCT ride_id) FROM RideStatus WHERE status = 'cancelled_by_driver') * 100.0 /
(SELECT COUNT(DISTINCT ride_id) FROM Rides) as driver_cancellation_rate,
(SELECT COUNT(DISTINCT ride_id) FROM RideStatus WHERE status = 'cancelled_by_rider') * 100.0 /
(SELECT COUNT(DISTINCT ride_id) FROM Rides) as rider_cancellation_rate;
```

9. **Impact of weather on ride demand**
```sql
SELECT w.weather_condition, COUNT(*) AS number_of_rides
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Weather w ON l.city_id = w.city_id
AND DATE_FORMAT(r.ride_date_time, '%Y-%m-%d %H:00:00') = w.date_time
GROUP BY w.weather_condition;
```

10. **Rider loyalty metrics**
```sql
SELECT 
    ri.rider_name, 
    COUNT(*) as total_rides, 
    AVG(r.fare) as avg_fare,
    SUM(r.fare) as total_spend,
    DATEDIFF(MAX(r.ride_date_time), ri.signup_date) as days_since_signup,
    COUNT(*)/NULLIF(DATEDIFF(MAX(r.ride_date_time), ri.signup_date), 0) as rides_per_day
FROM Rides r
JOIN Riders ri ON r.rider_id = ri.rider_id
GROUP BY ri.rider_name;
```

This data model provides a comprehensive foundation for analyzing Uber's operations and customer behavior while maintaining a clean star schema design. The RideStatus table is implemented as a transaction fact table to track all status changes over time, which provides more analytical flexibility than a dimension table approach.


<br/>
<br/>

# **Uber Data Analytics Platform: Sample Input & Output**

## **1. Sample Input Data (Raw Data Before Processing)**
### **A. Driver Registration (Dimension Table)**
```csv
driver_id,driver_name,driver_phone,driver_email,signup_date,vehicle_type,city_id
D001,John Doe,+1555123456,john@uber.com,2023-01-15,Sedan,C1
D002,Jane Smith,+1555234567,jane@uber.com,2023-02-20,SUV,C2
```

### **B. Rider Registration (Dimension Table)**
```csv
rider_id,rider_name,rider_phone,rider_email,signup_date,loyalty_status,city_id
R001,Alex Brown,+1555345678,alex@uber.com,2023-03-10,Gold,C1
R002,Sarah Lee,+1555456789,sarah@uber.com,2023-04-05,Silver,C2
```

### **C. Cities & Locations (Dimension Tables)**
```csv
# Cities
city_id,city_name,state,country
C1,New York,NY,USA
C2,Los Angeles,CA,USA

# Locations
location_id,city_id,location_name,latitude,longitude
L1,C1,Time Square,40.7580,-73.9855
L2,C1,Central Park,40.7829,-73.9654
L3,C2,Hollywood,34.0928,-118.3287
```

### **D. Weather Data (Dimension Table)**
```csv
weather_id,city_id,date_time,weather_condition,temperature
W1,C1,2023-05-01 08:00:00,Sunny,72
W2,C2,2023-05-01 08:00:00,Rainy,65
```

### **E. Ride Events (Fact Tables - Real-Time Stream)**
```json
// Ride Request
{
  "ride_id": "TR001",
  "rider_id": "R001",
  "start_location_id": "L1",
  "end_location_id": "L2",
  "request_time": "2023-05-01 08:05:00"
}

// Driver Accepts
{
  "ride_id": "TR001",
  "driver_id": "D001",
  "status": "accepted",
  "status_time": "2023-05-01 08:06:00"
}

// Ride Starts (Pickup)
{
  "ride_id": "TR001",
  "status": "started",
  "status_time": "2023-05-01 08:10:00"
}

// Ride Completes
{
  "ride_id": "TR001",
  "status": "completed",
  "status_time": "2023-05-01 08:25:00",
  "fare": 15.50,
  "ride_duration": 15, // minutes
  "ride_distance": 3.2, // miles
  "rating_by_driver": 5,
  "rating_by_rider": 4
}
```

---

## **2. Processed Output (Analytics Results)**
### **A. Business Metrics (Dashboard Visualizations)**
#### **1. Average Ride Duration & Distance Per City**
| City        | Avg Duration (mins) | Avg Distance (miles) |
|-------------|---------------------|----------------------|
| New York    | 12.5                | 2.8                  |
| Los Angeles | 18.2                | 5.1                  |

**Insight**: LA rides are longer due to urban sprawl.

#### **2. Revenue Per City (Last 7 Days)**
| City        | Revenue ($) |
|-------------|------------|
| New York    | 12,450     |
| Los Angeles | 8,920      |

**Insight**: NYC generates more revenue due to higher demand.

#### **3. Peak Ride Hours (NYC)**
| Hour | Number of Rides |
|------|-----------------|
| 8 AM | 1,250           |
| 5 PM | 1,480           |

**Insight**: Rush hours have the highest demand.

#### **4. Cancellation Rates**
| Cancelled By | Rate (%) |
|--------------|----------|
| Riders       | 8.2%     |
| Drivers      | 5.1%     |

**Insight**: Riders cancel more often (possibly due to long wait times).

#### **5. Weather Impact on Rides**
| Weather Condition | Rides Completed |
|-------------------|-----------------|
| Sunny             | 9,200           |
| Rainy             | 6,500           |

**Insight**: Rain reduces rides by ~30%.

---

### **B. Real-Time Alerts (Sample)**
1. **Surge Pricing Alert**  
   - *"High demand detected in NYC at 5 PM. Surge multiplier: 1.8x"*  

2. **Driver Cancellation Spike**  
   - *"Warning: Driver cancellations increased by 20% in LA today."*  

3. **Rider Loyalty Opportunity**  
   - *"Rider R001 has taken 10+ rides this month. Consider a discount offer!"*  

---

## **3. How Data Flows Through the System**
### **Step 1: Data Ingestion**
- **Mobile App Events** (ride requests, GPS updates) → **Kafka Topic `uber-rides`**  
- **Weather API** → **Batch Load (Daily) into `Weather` table**  

### **Step 2: Stream Processing (Apache Flink)**
```java
// Flink Job Pseudocode
DataStream<RideEvent> rideEvents = KafkaSource("uber-rides");

// Calculate time-to-pickup
rideEvents
  .keyBy("ride_id")
  .process(new RideStatusTracker())
  .sinkTo(PostgreSQL); // Updates RideStatus table
```

### **Step 3: Batch Processing (Airflow + Spark)**
```python
# Daily Aggregation Job (Spark SQL)
df = spark.sql("""
  SELECT city_id, AVG(ride_duration) 
  FROM Rides 
  GROUP BY city_id
""")
df.write_to_snowflake() # OLAP Storage
```

### **Step 4: Dashboard Updates (Tableau)**
- Queries Snowflake for latest aggregates.  
- Displays trends like:  
  - **"Revenue increased by 12% WoW in NYC"**  
  - **"Driver D001 has a 4.9/5 avg rating"**  

---

## **4. Key Insights from Sample Data**
1. **Driver Performance**  
   - John Doe (D001) took **5 mins** to pick up Alex Brown (fast!).  
   - Avg rating: **4.5/5** → Good candidate for bonuses.  

2. **Route Popularity**  
   - **Time Square → Central Park** is a frequent route (sample ride TR001).  

3. **Weather Correlation**  
   - Rain in LA (**65°F**) → Fewer rides than sunny NYC (**72°F**).  

4. **Loyalty Opportunities**  
   - Alex Brown (R001) is a **Gold** member → Target for promotions.  

---

## **5. Try It Yourself (Sample Queries)**
### **A. Find Top 5 Drivers by Rating**
```sql
SELECT d.driver_name, AVG(r.rating_by_rider) as avg_rating
FROM Rides r JOIN Drivers d ON r.driver_id = d.driver_id
GROUP BY d.driver_name
ORDER BY avg_rating DESC
LIMIT 5;
```

### **B. Rides Affected by Rain**
```sql
SELECT COUNT(*) as rainy_rides
FROM Rides r JOIN Weather w ON r.city_id = w.city_id
WHERE w.weather_condition = 'Rainy'
AND DATE(r.ride_date_time) = DATE(w.date_time);
```

### **C. Rider Lifetime Value (LTV)**
```sql
SELECT 
  ri.rider_name, 
  SUM(r.fare) as total_spend,
  COUNT(*) as total_rides
FROM Rides r JOIN Riders ri ON r.rider_id = ri.rider_id
GROUP BY ri.rider_name
ORDER BY total_spend DESC;
```

---

## **6. Conclusion**
This sample input/output demonstrates how raw ride events transform into actionable insights:
- **Real-time tracking** → "Driver took 5 mins to pick up rider."  
- **Batch analytics** → "LA has longer but fewer rides than NYC."  
- **Business impact** → Adjust pricing, reward top drivers, reduce cancellations.  


<br/>
<br/>

# **Detailed Explanation of Uber SQL Queries**

## **1. Average Ride Duration and Distance per City**
```sql
SELECT c.city_name, AVG(r.ride_duration), AVG(r.ride_distance)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Cities c ON l.city_id = c.city_id
GROUP BY c.city_name;
```

**Purpose**:  
Calculates the typical length (time and distance) of rides in each city.

**Key Components**:
- **AVG(ride_duration)**: Mean trip time in minutes
- **AVG(ride_distance)**: Mean trip distance in miles/km
- **City grouping**: Shows urban vs. suburban patterns

**Business Insight**:  
Helps optimize driver allocation (longer rides may need different pricing) and understand city traffic patterns.

---

## **2. Revenue Generated per City**
```sql
SELECT c.city_name, SUM(r.fare)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Cities c ON l.city_id = c.city_id
WHERE r.status = 'completed'
GROUP BY c.city_name;
```

**Purpose**:  
Measures total earnings from completed rides by city.

**Key Differences from Duration Query**:
- Uses `SUM` instead of `AVG`
- Filters only `completed` rides
- Focuses on financials rather than trip characteristics

**Business Insight**:  
Identifies most profitable markets for business expansion.

---

## **3. Peak Times for Rides**
```sql
SELECT l.location_name, d.hour, COUNT(*)
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Dates d ON r.ride_date_time = d.date_time
GROUP BY l.location_name, d.hour
ORDER BY COUNT(*) DESC;
```

**Purpose**:  
Identifies high-demand hours at specific locations.

**Key Components**:
- **Hourly grouping**: Breaks down demand by time of day
- **COUNT(*)**: Measures ride volume
- **Location-specific**: Shows where/when surges occur

**Business Insight**:  
Guides surge pricing strategies and driver positioning.

---

## **4. Popular Routes and Destinations**
```sql
SELECT l1.location_name AS start_location, l2.location_name AS end_location, COUNT(*) AS number_of_rides
FROM Rides r
JOIN Locations l1 ON r.start_location_id = l1.location_id
JOIN Locations l2 ON r.end_location_id = l2.location_id
GROUP BY l1.location_name, l2.location_name
ORDER BY number_of_rides DESC;
```

**Purpose**:  
Maps most frequently traveled origin-destination pairs.

**Special Features**:
- **Self-join** on Locations table (start vs. end points)
- **Route-based analysis**: More valuable than individual locations

**Business Insight**:  
Helps optimize navigation routes and identify potential shuttle routes.

---

## **5. Average Rider Rating per Driver**
```sql
SELECT d.driver_name, AVG(r.rating_by_driver)
FROM Rides r
JOIN Drivers d ON r.driver_id = d.driver_id
GROUP BY d.driver_name;
```

**Purpose**:  
Measures how riders rate different drivers.

**Rating System**:
- Typically 1-5 scale
- Reflects driver behavior, car condition, etc.

**Business Insight**:  
Used for driver bonuses/deactivations and quality control.

---

## **6. Average Driver Rating per Rider**
```sql
SELECT ri.rider_name, AVG(r.rating_by_rider)
FROM Rides r
JOIN Riders ri ON r.rider_id = ri.rider_id
GROUP BY ri.rider_name;
```

**Purpose**:  
Shows how drivers rate passengers.

**Key Difference from Query 5**:
- Rates riders instead of drivers
- Helps identify problematic passengers

**Business Insight**:  
May trigger rider warnings or bans for consistently low ratings.

---

## **7. Time from Acceptance to Pickup**
```sql
SELECT d.driver_id, d.driver_name, r.ride_id,
TIMEDIFF(pickup_status.time, acceptance_status.time) AS time_to_pickup
FROM RideStatus acceptance_status
JOIN RideStatus pickup_status ON acceptance_status.ride_id = pickup_status.ride_id
JOIN Rides r ON r.ride_id = acceptance_status.ride_id
JOIN Drivers d ON r.driver_id = d.driver_id
WHERE acceptance_status.status = 'accepted'
AND pickup_status.status = 'started';
```

**Purpose**:  
Measures driver efficiency in reaching customers.

**Complexity**:
- Requires **self-join** on RideStatus
- Calculates time delta between status changes

**Business Insight**:  
Identifies slow responders for additional training.

---

## **8. Ride Cancellation Rates**
```sql
SELECT
(SELECT COUNT(DISTINCT ride_id) FROM RideStatus WHERE status = 'cancelled_by_driver') * 1.0 /
(SELECT COUNT(DISTINCT ride_id) FROM Rides) as driver_cancellation_rate,
(SELECT COUNT(DISTINCT ride_id) FROM RideStatus WHERE status = 'cancelled_by_rider') * 1.0 /
(SELECT COUNT(DISTINCT ride_id) FROM Rides) as rider_cancellation_rate;
```

**Purpose**:  
Calculates percentage of rides cancelled by each party.

**Special Syntax**:
- Uses **subqueries** for separate calculations
- Multiplies by 1.0 for decimal results

**Business Insight**:  
High driver cancellations may indicate incentive issues; high rider cancellations may suggest pricing problems.

---

## **9. Weather Impact on Demand**
```sql
SELECT w.weather_condition, COUNT(*) AS number_of_rides
FROM Rides r
JOIN Locations l ON r.start_location_id = l.location_id
JOIN Weather w ON l.city_id = w.city_id
AND DATE_FORMAT(r.ride_date_time, '%Y-%m-%d %H:00:00') = w.date_time
GROUP BY w.weather_condition;
```

**Purpose**:  
Correlates ride volume with weather conditions.

**Key Join Condition**:
- Matches ride timestamp to hourly weather data

**Business Insight**:  
Helps predict demand surges during bad weather.

---

## **10. Rider Loyalty Metrics**
```sql
SELECT ri.rider_name, COUNT(*), AVG(r.fare), DATEDIFF(MAX(r.ride_date_time), ri.signup_date) as days_since_signup
FROM Rides r
JOIN Riders ri ON r.rider_id = ri.rider_id
GROUP BY ri.rider_name;
```

**Purpose**:  
Analyzes customer retention and spending habits.

**Metrics Calculated**:
1. Ride frequency (`COUNT`)
2. Average spend (`AVG`)
3. Customer tenure (`DATEDIFF`)

**Business Insight**:  
Identifies valuable customers for loyalty programs.

---

## **Common Analytical Patterns**
1. **Multi-table Joins**: Connecting rides to locations, drivers, weather, etc.
2. **Time Intelligence**: Hourly/daily/monthly aggregations
3. **Ratio Metrics**: Cancellation rates, average ratings
4. **Behavioral Analysis**: Rider/driver patterns
5. **Environmental Factors**: Weather impact

These queries collectively optimize:
- **Operations** (driver allocation, surge pricing)
- **Customer Experience** (ratings, cancellations)
- **Business Strategy** (market expansion, loyalty programs)