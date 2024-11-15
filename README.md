# Currency Data Processing and Analysis Pipeline  

This repository showcases a robust data pipeline built with Apache NiFi for fetching, processing, storing, and analyzing currency exchange data. The system integrates Apache Cassandra for scalable data storage and Grafana for real-time visualization, providing insights into currency trends over time.  

---

## 📋 Project Overview  

The **Currency Data Processing and Analysis Pipeline** streamlines the continuous ingestion and analysis of currency exchange data from APIs. This pipeline is designed for scalability, allowing easy expansion to support additional currencies or other financial datasets.  

---

## 🚀 Features  

- **Real-Time Data Ingestion**: Simultaneous retrieval of multiple currency pairs or conversion rates.  
- **Incremental Data Updates**: Automated date handling to simulate daily feeds.  
- **Customizable Data Transformation**: Flexible JSON parsing and data formatting for storage.  
- **Scalable Storage**: Apache Cassandra ensures efficient storage and query capabilities for time-series data.  
- **Intuitive Visualization**: Grafana dashboards provide real-time analytics for actionable insights.  

---

## 🖥️ System Architecture Diagram 

![Project Architecture Diagram](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/Project%20Diagram.gif)

---

## 🛠️ Project Workflow  

### 1. **Data Ingestion with Apache NiFi**  
- Utilize **InvokeHTTP** processors to call API endpoints for various currency pairs or rates.  
- Fetch JSON-formatted data in real-time, which is passed through the pipeline for processing.  

### 2. **Date Handling and Incremental Updates**  
- Employ **GenerateFlowFile** to initialize a starting date (e.g., `1999-01-04`).  
- Increment dates using **UpdateAttribute** and recursive looping to simulate daily data feeds.  

### 3. **Data Transformation**  
- Extract key fields (e.g., currency codes, exchange rates, timestamps) using **EvaluateJsonPath**.  
- Reformat data for storage using **UpdateAttribute** or **ReplaceText** processors.  

### 4. **Data Storage in Apache Cassandra**  
- Insert processed data into Cassandra using **PutCassandraQL**.  
- Store records with attributes such as:  
  - **Currency Pair**  
  - **Exchange Rate**  
  - **Timestamp**  

### 5. **Real-Time Visualization with Grafana**  
- Connect Apache Cassandra to Grafana to create customizable dashboards.  
- Perform:  
  - Trend analysis  
  - Rate comparisons  
  - Insightful visualizations of currency fluctuations  

---

### Key Components  

| Component            | Purpose                                                                 |  
|-----------------------|-------------------------------------------------------------------------|  
| **Apache NiFi**       | Handles data ingestion, transformation, and orchestration for the pipeline. |  
| **Apache Cassandra**  | Provides scalable, highly available storage for time-series data.       |  
| **Grafana**           | Enables visualization and real-time analytics through customizable dashboards. |  

---

## 🔧 Setup  

### Prerequisites  
- Apache NiFi  
- Apache Cassandra  
- Grafana  

### NiFi Setup
![NiFi Diagram](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/Nifi_Diagram.png)


## 1. **GenerateFlowFile**  
- **Purpose**: Generates a flowfile containing a date attribute, starting from `1999-01-04`.  
- **Configuration**:  
  - Set to create a flowfile with an initial date to simulate historical currency data feeds.  
  - This serves as the starting point for querying data by day.  

  ![GenerateFlowFile](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/GenerateFlowFile.png)

---  

## 2. **UpdateAttribute**  
- **Purpose**: Updates the date by incrementing it by one day, creating a recursive loop to fetch daily data.  
- **Configuration**:  
  - Uses NiFi Expression Language to add one day:  
    ```  
    ${Current_Date:toDate("yyyy-MM-dd"):toNumber():plus(86400000):toDate():format("yyyy-MM-dd")}  
    ```  
  - Ensures the date advances until the desired end date.  

  ![UpdateAttribute](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/UpdateAttribute.png)

---  

## 3. **InvokeHTTP (API Call for Different Currencies)**  
- **Purpose**: Fetches exchange rates for different currencies (e.g., INR, EUR, USD, etc.).  
- **Configuration**:  
  - Dynamically sets URL parameters for each currency base with the incremented date.  

  ![InvokeHTTP](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/InvokeHTTP.png)

---  

## 4. **EvaluateJsonPath**  
- **Purpose**: Extracts relevant data from the API response JSON.  
- **Configuration**:  
  - Extracts fields like base currency, date, and exchange rates (e.g., AUD, GBP).  

  ![EvaluateJsonPath](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/EvaluateJSONPath.png)

---  

## 5. **UpdateAttribute (Null Replacement)**  
- **Purpose**: Prepares data for Cassandra by replacing empty or missing values with null.  
- **Configuration**:  
  - Ensures compatibility with Cassandra schema to avoid errors for missing data.  

  ![UpdateAttribute](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/UpdateAttribute_2.png)

---  

## 6. **ReplaceText**  
- **Purpose**: Constructs an `INSERT` query in CQL for inserting data into Cassandra.  
- **Configuration**:  
  - Uses placeholders replaced with actual attribute values for accuracy:  
    ```  
    INSERT INTO bigdata.currency ( "Date", "Base" , "Amount", AUD, BGN, ... )  
    VALUES ('${Date}', '${Base}', ${Amount}, ${AUD}, ${BGN}, ... );  
    ```  

  ![ReplaceText](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/ReplaceText.png)

---  

## 7. **PutCassandraQL**  
- **Purpose**: Executes the CQL query, storing processed currency data in Cassandra.  
- **Configuration**:  
  - Points to the `bigdata.currency` table with the following schema:  
    ```  
    CREATE TABLE bigdata.currency (  
      "Date" date,  
      "Base" text,  
      "Amount" float,  
      AUD float, BGN float, ...,  
      PRIMARY KEY ("Base", "Date")  
    );  
    ```  

  ![PutCassandraQL](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/PutCassandraQL.png)


## Notes  

- **Date Increment Handling**: The `UpdateAttribute` processor ensures a continuous loop for daily data.  
- **Error Handling**: Monitor error queues and logs in `InvokeHTTP` and `PutCassandraQL`.  
- **Performance Optimization**: Tune NiFi’s flowfile queuing and processor execution times for scalability.  


---  

## Cassandra Setup  

### **Keyspace Creation**  
- The following command was used to create the keyspace for storing currency data:
  ```
  CREATE KEYSPACE IF NOT EXISTS bigdata
  WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' };
  ```

### **Table Schema**  
- The `currency` table schema ensures scalable and efficient storage for daily exchange rate data:  
  ```
  CREATE TABLE IF NOT EXISTS currency ( "Date" DATE,
  "Base" TEXT,
  "Amount" float, AUD float, BGN float, BRL float, 
  CAD float, CHF float, CNY float,  CZK float, 
  DKK float, EUR float, GBP float, HKD float,
  HUF float, IDR float, ILS float, INR float, 
  ISK float, JPY float, KRW float, MXN float, 
  MYR float, NOK float, NZD float, PHP float, 
  PLN float, RON float, SEK float, SGD float,
  THB float, TRY float, USD float, ZAR float,
  PRIMARY KEY (("Base"), "Date"));
  ```
  ![Cassandra View](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/Cassandra_DB.png)  

## Analysis and Visualization  

### **Grafana Integration**  
- **Purpose**: Visualize trends using data stored in Cassandra.  
- **Details**:  
  - Queries the `bigdata.currency` table for insights like daily changes or currency comparisons.  
  - Provides customizable dashboards to track volatility and trends.  

---  

## 📊 Sample Dashboard  

![Grafana Dashboard](https://github.com/WadyOsama/Currency-Data-Processing-and-Analysis-Pipeline/blob/main/Photos/Grafana_Dashboard.png)  
*Example of currency trend analysis in Grafana.*  

---

## 📈 Future Enhancements  

- Add support for additional APIs and financial data sources.  
- Implement alerting mechanisms for significant currency fluctuations.  
- Extend the pipeline to support real-time data analysis.  

---

## 📧 Contact  

For questions or feedback, please open an issue or contact:  
- **Name:** Wady Osama  
- **Email:** wady.alazb@gmail.com
- **Name:** Mohamed Zaghloul
- **Email:** mohamedzaghloul199@gmail.com 

---  

### ⭐ If you found this project useful, don't forget to give it a star!
