https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
# Data Pipeline
![[Pasted image 20250203011506.png]]

---
# TLC Trip Record Data

Yellow and green taxi trip records include fields capturing pick-up and drop-off dates/times, pick-up and drop-off locations, trip distances, itemized fares, rate types, payment types, and driver-reported passenger counts. The data used in the attached datasets were collected and provided to the NYC Taxi and Limousine Commission (TLC) by technology providers authorized under the Taxicab & Livery Passenger Enhancement Programs (TPEP/LPEP). The trip data was not created by the TLC, and TLC makes no representations as to the accuracy of these data.

For-Hire Vehicle (“FHV”) trip records include fields capturing the dispatching base license number and the pick-up date, time, and taxi zone location ID (shape file below). These records are generated from the FHV Trip Record submissions made by bases. Note: The TLC publishes base trip record data as submitted by the bases, and we cannot guarantee or confirm their accuracy or completeness. Therefore, this may not represent the total amount of trips dispatched by all TLC-licensed bases. The TLC performs routine reviews of the records and takes enforcement actions when necessary to ensure, to the extent possible, complete and accurate information.

This is a comprehensive transcript of the video, covering an **end-to-end Uber Data Engineering project** that involves:
1. **Dataset Selection & Data Modeling**
    - Using an Uber-like dataset
    - Building a **fact and dimension data model**
    - Writing **data transformation code in Python**
2. **Cloud Deployment**
    - Deploying the transformation code on a **Google Cloud Compute Instance**
    - Installing **[[Mage]]**, an **open-source data pipeline tool**
    - Loading the data into **Google BigQuery** (a cloud-based data warehouse)
3. **Final Dashboard**
    - Creating a **dashboard using Looker Studio (formerly Data Studio)**
    - Building visualizations and insights from transformed data

### **Step-by-Step Breakdown**

#### **1. Data Architecture**
- Data is stored on **Google Cloud Storage** (GCS)
- Data transformation logic is written in **Python**
- Mage is deployed on **Google Compute Engine**
- Transformed data is stored in **BigQuery**
- Final dashboard is built using **Looker Studio**

#### **2. Google Cloud Services Used**
- **Google Cloud Storage (GCS)**: Object storage (similar to AWS S3, Azure Blob)
- **Compute Engine**: Virtual Machine (similar to AWS EC2, Azure VM)
- **BigQuery**: Cloud-based Data Warehouse for large-scale data analysis
- **Looker Studio**: BI tool for visualizing data (similar to Tableau, Power BI)

#### **3. Data Pipeline Execution**
- **Step 1:** Store data in **Google Cloud Storage (GCS)**
- **Step 2:** Deploy **Mage** on Google Compute Engine
- **Step 3:** Write **data transformation logic** in Python using Pandas
- **Step 4:** Load transformed data into **BigQuery**
- **Step 5:** Build a **dashboard in Looker Studio**

#### **4. Understanding Fact & Dimension Tables**
- **Fact Table**: Stores transactional values (e.g., Order ID, Total Revenue)
- **Dimension Table**: Stores descriptive attributes (e.g., Product Name, Customer Details)

**Example in Uber Dataset:**

- **Fact Table**: `trip_id`, `fare_amount`, `pickup_time`, `dropoff_time`
- **Dimension Tables**: `passenger_dim`, `rate_code_dim`, `payment_dim`, `location_dim`

#### **5. Google Cloud Implementation**
- Upload dataset to **Google Cloud Storage**
- Deploy a **Compute Engine instance** for running Python scripts
- Install **Mage**, a modern ETL tool, to automate the pipeline
- Use **BigQuery** to store and query transformed data
- Create **SQL queries** to analyze ride data

#### **6. Building the Final Dashboard**
- **Filters**: Vendor ID, Payment Type, Rate Code
- **Summary Metrics**: Total Revenue, Average Fare, Total Trips
- **Maps**: Displaying pickup & drop-off locations
- **Bar Charts**: Breakdown of revenue by payment type & rate code
- **Line Graphs**: Trends in ride counts over time

#### **7. Key SQL Queries in BigQuery**
- **Finding the most popular pickup locations**
- **Average fare amount per vendor**
- **Total trip count by passenger count**
- **Payment type distribution for Uber trips**
- **Revenue distribution by rate codes**

---
# Dimensionality Modelling:
![[Pasted image 20250201233103.png]]
### Key Characteristics of a Star Schema in the Diagram:
1. **Central Fact Table**:
    - The **Fact Table** contains the measurable and transactional data (e.g., `fare_amount`, `tip_amount`, `total_amount`) with foreign keys pointing to dimension tables. This aligns with the definition of a star schema.
2. **Dimension Tables**:
    - Dimension tables (e.g., `DateTime_dim`, `pickup_location_dim`, `dropoff_location_dim`) provide descriptive, contextual data for the fact table.
    - Each dimension table is denormalized (e.g., `DateTime_dim` has attributes like `pick_hour`, `pick_day`, etc.) to improve query performance.
3. **Single-Level Relationships**:
    - Each dimension table directly connects to the fact table via foreign keys (e.g., `datetime_id`, `ratecode_id`). There are no additional hierarchies or dependencies between dimensions, which is a hallmark of a star schema.
4. **Simplified Structure**:
    - The schema is easy to interpret and optimized for analytical queries, enabling efficient aggregation and reporting.
### Why It’s a Star Schema:
- The **Fact Table** serves as the hub, and the **Dimension Tables** act as the spokes, creating the "star" shape when visualized.
- Each dimension table contains descriptive attributes, and there’s no further normalization (as in a snowflake schema).
### Use Cases:
This schema is ideal for:
- Business intelligence tools like Power BI or Tableau.
- Queries requiring filtering, grouping, and aggregation across dimensions (e.g., analyzing revenue by time, location, or passenger count).

# Python transformation code for dimensionality Modelling
```
import pandas as pd
df = pd.read_csv("uber_data.csv")

# converting to datetime
df['tpep_pickup_datetime'] = pd.to_datetime(df['tpep_pickup_datetime'])
df['tpep_dropoff_datetime'] = pd.to_datetime(df['tpep_dropoff_datetime'])
df.drop_duplicates().reset_index(drop = True)
df['trip_id'] = df.index

# creating a datetime dimension table

datetime_dim = df[['tpep_pickup_datetime','tpep_dropoff_datetime']].reset_index(drop = True)
datetime_dim['tpep_pickup_datetime'] = datetime_dim['tpep_pickup_datetime']
datetime_dim['pick_hour'] = datetime_dim['tpep_pickup_datetime'].dt.hour
datetime_dim['pick_day'] = datetime_dim['tpep_pickup_datetime'].dt.day
datetime_dim['pick_month'] = datetime_dim['tpep_pickup_datetime'].dt.month
datetime_dim['pick_year'] = datetime_dim['tpep_pickup_datetime'].dt.year
datetime_dim['pick_weekday'] = datetime_dim['tpep_pickup_datetime'].dt.weekday

datetime_dim['tpep_dropoff_datetime'] = datetime_dim['tpep_dropoff_datetime']
datetime_dim['drop_hour'] = datetime_dim['tpep_dropoff_datetime'].dt.hour
datetime_dim['drop_day'] = datetime_dim['tpep_dropoff_datetime'].dt.day
datetime_dim['drop_month'] = datetime_dim['tpep_dropoff_datetime'].dt.month
datetime_dim['drop_year'] = datetime_dim['tpep_dropoff_datetime'].dt.year
datetime_dim['drop_weekday'] = datetime_dim['tpep_dropoff_datetime'].dt.weekday


datetime_dim['datetime_id'] = datetime_dim.index

# datetime_dim = datetime_dim.rename(columns={'tpep_pickup_datetime': 'datetime_id'}).reset_index(drop=True)
datetime_dim = datetime_dim[['datetime_id', 'tpep_pickup_datetime', 'pick_hour', 'pick_day', 'pick_month', 'pick_year', 'pick_weekday',
                             'tpep_dropoff_datetime', 'drop_hour', 'drop_day', 'drop_month', 'drop_year', 'drop_weekday']]

datetime_dim

passenger_count_dim = df[['passenger_count']].reset_index(drop=True)
passenger_count_dim['passenger_count_id'] = passenger_count_dim.index
passenger_count_dim = passenger_count_dim[['passenger_count_id','passenger_count']]

trip_distance_dim = df[['trip_distance']].reset_index(drop=True)
trip_distance_dim['trip_distance_id'] = trip_distance_dim.index
trip_distance_dim = trip_distance_dim[['trip_distance_id','trip_distance']]

rate_code_type = {
    1:"Standard rate",
    2:"JFK",
    3:"Newark",
    4:"Nassau or Westchester",
    5:"Negotiated fare",
    6:"Group ride"
}

rate_code_dim = df[['RatecodeID']].reset_index(drop=True)
rate_code_dim['rate_code_id'] = rate_code_dim.index
rate_code_dim['rate_code_name'] = rate_code_dim['RatecodeID'].map(rate_code_type)
rate_code_dim = rate_code_dim[['rate_code_id','RatecodeID','rate_code_name']]

pickup_location_dim = df[['pickup_longitude', 'pickup_latitude']].reset_index(drop=True)
pickup_location_dim['pickup_location_id'] = pickup_location_dim.index
pickup_location_dim = pickup_location_dim[['pickup_location_id','pickup_latitude','pickup_longitude']] 

dropoff_location_dim = df[['dropoff_longitude', 'dropoff_latitude']].reset_index(drop=True)
dropoff_location_dim['dropoff_location_id'] = dropoff_location_dim.index
dropoff_location_dim = dropoff_location_dim[['dropoff_location_id','dropoff_latitude','dropoff_longitude']]

payment_type_name = {
    1:"Credit card",
    2:"Cash",
    3:"No charge",
    4:"Dispute",
    5:"Unknown",
    6:"Voided trip"
}
payment_type_dim = df[['payment_type']].reset_index(drop=True)
payment_type_dim['payment_type_id'] = payment_type_dim.index
payment_type_dim['payment_type_name'] = payment_type_dim['payment_type'].map(payment_type_name)
payment_type_dim = payment_type_dim[['payment_type_id','payment_type','payment_type_name']]

fact_table = df.merge(passenger_count_dim, left_on='trip_id', right_on='passenger_count_id') \
             .merge(trip_distance_dim, left_on='trip_id', right_on='trip_distance_id') \
             .merge(rate_code_dim, left_on='trip_id', right_on='rate_code_id') \
             .merge(pickup_location_dim, left_on='trip_id', right_on='pickup_location_id') \
             .merge(dropoff_location_dim, left_on='trip_id', right_on='dropoff_location_id')\
             .merge(datetime_dim, left_on='trip_id', right_on='datetime_id') \
             .merge(payment_type_dim, left_on='trip_id', right_on='payment_type_id') \
             [['trip_id','VendorID', 'datetime_id', 'passenger_count_id',
               'trip_distance_id', 'rate_code_id', 'store_and_fwd_flag', 'pickup_location_id', 'dropoff_location_id',
               'payment_type_id', 'fare_amount', 'extra', 'mta_tax', 'tip_amount', 'tolls_amount',
               'improvement_surcharge', 'total_amount']]

fact_table
```
---
# Shifting data to GCP 

Created a bucket using google cloud console
Changed the bucket access to public so that the code can access the dataset

URL to download: https://storage.googleapis.com/uber_data_csv/uber_data.csv

Created a GCP compute instance: instance-uberdata
### In AWS we have to download the keys and then connect to ssh but in gcp the connection is streamlined by directly using the ssh button given on the instance info bar.

Running following commands in the ssh terminal
```
sudo apt-get update -y
sudo apt-get install python3-distutils
sudo apt-get install python3-apt
sudo apt-get install wget
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```

Instead of the last command to get pip I had to use this instead:
```
sudo rm /usr/lib/python3.11/EXTERNALLY-MANAGED
```
#### Installing Mage
```
pip install mage-ai
```
#### Starting a Mage project:
```
mage start uber_de_project
```
#### Lookout for:
```
:Mage is running at http://localhost:6789 and serving project /home/arya_apps14/uber_de_project
```

The instance firewall rules are not allowing to connect using the port 6789, so I will add a new rule which allows connections using this port

`ip added: 0.0.0.0/0`

The /0 allows us to access the instance from anywhere.

I have kept the firewall public, for trials sake and added port 6789 to connect.
[Pipelines | Mage](http://34.55.182.255:6789/pipelines?_limit=30)
![[Pasted image 20250203015135.png]]

Creating a new standard pipeline called Uber_data_trial

Copy pasted the python code into mage transformation block
Make sure about the indendation

Now I will export the data to bigQuery

yaml file credentials:
```
{

  "type": "service_account",

  "project_id": "uberdataengineering-222025",

  "private_key_id": "1bba7c1403b5d0364a24307f50111af42ad7c4ae",

  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCzDZO5xA+4rUrk\n4ohcw5+DpTpej2ZL7MFehn05PHerqrui53JVZqV2dATmkjAYdQPkqvF1uagone6w\ni9BlXepytuIyPLIJ2EgGz+cRFJK9boKfEm8RU/EuCAqQyENQrlC5nNiV2AGlsnsy\nHbBRUn1JTWI7Mmzm6T36Xe/cwTqF4NkmG2e/FmX7Uj7Ue5i/3dSVqUFotBwfUTj/\nG69ABhu91jI/GZ6bp9xevXUpkig0E0dFPsGJyFqwlAa5IBcgngLInStmbqTcIOUZ\nSqJnMwEFmCKgmQa/C67RU5/tU7X8gKqpNSUX+sH9dgGk/gHm4xy14wUy03UYQDAC\npQc19uPvAgMBAAECggEAAy54FAJwVxM5M/T0gFV3haLPdTOuGSZFT1Urb3NsGvBH\nMj9TkEHpKcApACHT2fNmVM9WyAU7ADHCNn/dfZecHVqzjTn33eXqbbiO/gY0D0qh\n5oqwz4mCzRMWgPkV/R6Dz7CCRrNWYCfaOYs+gUtqb32BA9VDCx1U3RiDNRR34j4T\nb5aVvNFWaOhPhhfoUnMAxZZMdXH3UlEU8Ti316WXy+i4NwKCRmbKSd/xaQXnE4LZ\nrHz4lvr36hyhuLefhBdxN9CwubH6YtZDPrJZaTm4BQ1b8WAmNEU3YNTPIgjXbc22\nwcOrQYy3zcXL06/Iba6GtiKfIYjUdr8SEwEigavE+QKBgQDg0hLmthVH19WxkxJv\nJO1rSu8aN11GZ4Qapm7vuc/BjefOIkI/rAlB/3QBhKWhbVr+YkDWewD9/v770KQG\np/Tl+jweqfylIKtqQ9Wg1ACUjm81rsS7uNti116EzilSJa+O6wSBG1uk75zBb3As\nvzeDbXRW3yJbH10HEoOmnFfLKQKBgQDL4pfEG1gfg4G7qYTFR/05ILkLOEfTyYT0\ngr0AhAc1saGkCOWY/h1MEvPfXyq37cHOlT9plaLISUJrq2d1VElqWhsF1h8aQ2Ux\noxf90kL3rCrLlneBHyhZmIdOKIOm24MNkXgxZ1un4hRBT+kKJLZk7SB4br2FyRC0\noLksGg4xVwKBgGxZslRQkucCBXJEgFDiii34ek23OxPwVcGGTtboRVFyM7Kr3iPT\npM6S7/S/Whf3nTAWaEs2Et9W4sq33iV7EtM3i3v3ztRCb3qSYMXXBsSR9NT5esVC\nLTFwvJPizBVUJk0JxSz3424VMQYkz/ow2e9UdApeFa+26N28tYg5tzIpAoGATudz\nmqaGdTO8unbBdmQE4N4EHw555cHAnawXHyL5c1M6XjQ/PvVhOza+gLzg2GvALIXg\nE6mgZOjNNsQP9v1WqD0U5i5WvSBGAm6+8zEzT4ymx0GFIEiBoiMAgkP1p1aeolqg\n7GW2uAMrmZcmdhF9MRQAE/uvhx4oQ+9LmoC+62MCgYBY/9ND0gMp1EKeD+6LR6FM\nw2+D2wDQvAokmo6tZFjdEv9RJep4tVw2yrGLZ55CxqBfillkr54aMA2XX/lrQSl7\nAwCaw2GBKVzhtOB/p3CP4aiZoHYoWyryW1dJoqA+z3fe4Y3W7V5eXuCmzBeg0hGO\nUYbjFOHk0+AvdZ16NZPV7w==\n-----END PRIVATE KEY-----\n",

  "client_email": "uber-data-engineering@uberdataengineering-222025.iam.gserviceaccount.com",

  "client_id": "113168080667060124391",

  "auth_uri": "https://accounts.google.com/o/oauth2/auth",

  "token_uri": "https://oauth2.googleapis.com/token",

  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",

  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/uber-data-engineering%40uberdataengineering-222025.iam.gserviceaccount.com",

  "universe_domain": "googleapis.com"

}
```

Updated the exported code to add the big query dataset name and table name:
```
table_id = 'uberdataengineering-222025.uberDataEngineering.fact_table'
```

Using this piece of code to dynamically add tables to the bigquery dataset:
```
for key, value in data.items():

        table_id = 'uberdataengineering-222025.uberDataEngineering.{}'.format(key)

        BigQuery.with_config(ConfigFileLoader(config_path, config_profile)).export(

                DataFrame(value),

                table_id,

                if_exists='replace',  # Specify resolution policy if table name already exists

            )
```

---
# Big Query

Created a dataset called "uberDataEngineering"

Added all the tables using the method stated above

Now, I can run SQL queries in BigQuery
![[Pasted image 20250203145519.png]]
This query fetches avg tip amount by payment method type
```
select b.payment_type_name, avg(a.tip_amount) from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.payment_type_dim b

ON a.payment_type_id = b.payment_type_id

group by b.payment_type_name
```

# SQL Questions:
- What is the average tip amount for each payment type?
```
select b.payment_type_name, avg(a.tip_amount) from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.payment_type_dim b

ON a.payment_type_id = b.payment_type_id

group by b.payment_type_name
```
![[Pasted image 20250203151947.png]]
- Find the top 10 pickup locations based on the number of trips 
```
select b.pickup_latitude, b.pickup_longitude, COUNT(*) AS trip_count

from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.pickup_location_dim b

ON a.pickup_location_id = b.pickup_location_id

group by b.pickup_latitude, b.pickup_longitude

order by trip_count desc

limit 10
```
![[Pasted image 20250203151803.png]]
- Find the total number of trips by passenger count 
```
select b.passenger_count, count(*) as Total_Trips

from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.passenger_count_dim b

on a.passenger_count_id = b.passenger_count_id

group by b.passenger_count

order by Total_Trips desc
```
![[Pasted image 20250203151623.png]]
- Find the average fare amount by hour of the day
```
select avg(a.fare_amount) as Average_Fare, b.pick_hour as Pick_hour

from uberdataengineering-222025.uberDataEngineering.fact_table a

join uberdataengineering-222025.uberDataEngineering.datetime_dim b

on a.datetime_id = b.datetime_id

group by b.pick_hour

order by Pick_hour asc
```
![[Pasted image 20250203152804.png]]

Joining all the relevant columns:
```
SELECT

   f.VendorID,

   d.tpep_pickup_datetime,

   d.tpep_dropoff_datetime,

   p.passenger_count,

   td.trip_distance,

   rc.RatecodeID,

   f.store_and_fwd_flag,

   pl.pickup_latitude,

   pl.pickup_longitude,

  dl.dropoff_latitude,

   dl.dropoff_longitude,

   pt.payment_type,

   f.fare_amount,

   f.extra,

   f.mta_tax,

  f.tip_amount,

   f.tolls_amount,

   f.improvement_surcharge,

   f.total_amount

 FROM

   `uberdataengineering-222025.uberDataEngineering.fact_table` f

   JOIN `uberdataengineering-222025.uberDataEngineering.datetime_dim` d ON f.datetime_id = d.datetime_id

   JOIN `uberdataengineering-222025.uberDataEngineering.passenger_count_dim` p ON f.passenger_count_id = p.passenger_count_id

   JOIN `uberdataengineering-222025.uberDataEngineering.trip_distance_dim` td ON f.trip_distance_id = td.trip_distance_id

   JOIN `uberdataengineering-222025.uberDataEngineering.rate_code_dim` rc ON f.rate_code_id = rc.rate_code_id

  JOIN `uberdataengineering-222025.uberDataEngineering.pickup_location_dim` pl ON f.pickup_location_id = pl.pickup_location_id

   JOIN `uberdataengineering-222025.uberDataEngineering.dropoff_location_dim` dl ON f.dropoff_location_id = dl.dropoff_location_id

   JOIN `uberdataengineering-222025.uberDataEngineering.payment_type_dim` pt ON f.payment_type_id = pt.payment_type_id;
```

Data Snapshot:
![[Pasted image 20250204002114.png]]

---
# Dashboarding

For dashboarding I will use Looker Studio.
https://lookerstudio.google.com/reporting/f4b15d90-cbd9-4b21-8f1f-b05691c3fe8a
![[Pasted image 20250204001823.png]]
