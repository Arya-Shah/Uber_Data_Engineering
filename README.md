# 🚖 Uber Data Engineering & Analytics Pipeline  

## **Overview**  
This project showcases a **complete end-to-end data engineering pipeline** built to process and analyze Uber-like ride-sharing data. The pipeline integrates **Google Cloud Services (GCS, BigQuery, Looker Studio)** with **Mage AI** for ETL, transforming raw trip data into an optimized **star schema** for efficient querying and visualization.  

The **final output** is a **fully interactive dashboard** in **Looker Studio**, displaying key ride insights such as total revenue, trip distribution, fare trends, and geospatial analysis using a **geo bubble map**.

---

## **🚀 Tech Stack & Tools**
- **Mage AI**: Modern ETL orchestration tool
- ![Screenshot 2025-02-03 011448](https://github.com/user-attachments/assets/59550031-fb35-45a6-bcc8-ea326d800aef)
- **Google Cloud Services**:  
  - **Google Cloud Storage (GCS)** → Stores raw ride data
  - <img width="953" alt="Pasted image 20250204002114" src="https://github.com/user-attachments/assets/bc4f5f45-2117-494a-abbd-ca7b40ffd9e6" />
  - **BigQuery** → Optimized storage & querying
  - <img width="482" alt="Pasted image 20250203151623" src="https://github.com/user-attachments/assets/3dc5c959-7818-4a4c-bb97-b0166c8c6cd0" />
  - **Compute Engine** → Runs ETL pipelines  
- **Python (Pandas, SQLAlchemy)** → Data transformation  
- **SQL (BigQuery SQL)** → Querying transformed data  
- **Looker Studio** → Interactive dashboard
- <img width="601" alt="Pasted image 20250204001823" src="https://github.com/user-attachments/assets/922fd287-7584-4507-b114-2fbff2e47904" />
---
## **📌 Project Workflow**
### **1️⃣ Data Ingestion**
- Raw **NYC TLC trip records** are pulled from **Google Cloud Storage**.
- Data includes **pickup/dropoff times, locations, fare details, passenger counts, and payment types**.

### **2️⃣ Data Transformation & Star Schema**
- **ETL pipeline built using Mage AI & Python**:
  - Converts timestamps to structured datetime fields.
  - Creates **fact & dimension tables** (passenger count, trip distance, payment type, rate codes).
  - Geospatial processing for pickup & drop-off locations.
- **Schema Design (Star Schema)**:
  - **Fact Table:** Stores transactional ride details.
  - **Dimension Tables:** Provide descriptive attributes.

### **3️⃣ Data Loading & Storage**
- **BigQuery** serves as the centralized **data warehouse**.
- Optimized SQL queries allow **fast analytics**.

### **4️⃣ Data Analysis & Insights**
- SQL queries extract:
  - 🚖 **Top pickup locations**  
  - ⏳ **Trip frequency by time of day**  
  - 💰 **Average fare & tip trends**  
  - 📊 **Revenue distribution by payment type**  

### **5️⃣ Data Visualization**
- **Looker Studio Dashboard** (Google Data Studio)
  - 📌 **Geo Bubble Map** → Shows ride density across NYC  
  - 📈 **Bar & Pie Charts** → Fare trends, passenger distributions  
  - 📊 **Key Metrics** → Revenue, trip counts, average fares  
---

## **🔍 Key SQL Queries**
### **1. Top 10 Pickup Locations**
```sql
SELECT 
    b.pickup_latitude, 
    b.pickup_longitude, 
    COUNT(*) AS trip_count
FROM 
    `uberdataengineering.fact_table` a
JOIN 
    `uberdataengineering.pickup_location_dim` b
ON 
    a.pickup_location_id = b.pickup_location_id
GROUP BY 
    b.pickup_latitude, b.pickup_longitude
ORDER BY 
    trip_count DESC
LIMIT 10;

```

### **2. Find the total number of trips by passenger count **
```sql
select b.passenger_count, count(*) as Total_Trips

from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.passenger_count_dim b

on a.passenger_count_id = b.passenger_count_id

group by b.passenger_count

order by Total_Trips desc

```
### **3. Find the average fare amount by hour of the day**
```sql
select avg(a.fare_amount) as Average_Fare, b.pick_hour as Pick_hour

from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.datetime_dim b

on a.datetime_id = b.datetime_id

group by b.pick_hour

order by Pick_hour asc

```
### **4. What is the average tip amount for each payment type?**
```sql
select b.payment_type_name, avg(a.tip_amount) from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.payment_type_dim b

ON a.payment_type_id = b.payment_type_id

group by b.payment_type_name

```
---

## 📜 Lessons Learned

### ✅ Successes
- Successfully developed an **end-to-end data pipeline**, integrating **ETL, cloud storage, and data visualization**.
- Optimized **query performance** using a **star schema model** for analytical workloads.
- Delivered an **interactive dashboard** for **quick and meaningful insights**.

### ⚠️ Challenges
- Managing **large datasets** in **Looker Studio** required **pre-aggregating data** to avoid performance issues.
- Addressing **missing or null values** (e.g., latitude/longitude) required thorough **data cleaning**.
- Ensuring **seamless integration** across multiple **Google Cloud services** required **fine-tuned configuration**.
