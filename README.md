## **Project Overview**

**Title:** **Analysis of Web Server Logs using AWS and Apache Spark**

**Objective:** Build an ETL pipeline that processes web server log files to extract meaningful insights about user behavior, error rates, and performance metrics.

**Technologies Used:**

- **AWS Services:** S3, Glue, EMR, Lambda
- **Programming Languages:** Python, SQL
- **Big Data Technologies:** Apache Spark, SparkSQL, PySpark
- **Others:** Regular Expressions for log parsing

---

## **Project Architecture**

1. **Data Ingestion:**
   - Store raw web server log files in an Amazon S3 bucket.

2. **Data Processing:**
   - Use AWS Glue or AWS Lambda with PySpark to process the log files.
   - Use Regular Expressions to parse log entries.

3. **Data Transformation:**
   - Convert raw logs into structured data frames.
   - Perform transformations using SparkSQL.

4. **Data Storage:**
   - Store the processed data back into S3 in Parquet format for efficient querying.

5. **Data Analysis:**
   - Use SparkSQL to run queries and generate reports.

6. **Visualization (Optional):**
   - Use AWS QuickSight or Jupyter Notebooks to visualize the data.

---

## **Step-by-Step Implementation Guide**

### **1. Setting Up AWS Resources**

- **Create an S3 Bucket:**
  - Store raw and processed data.
- **Set Up IAM Roles:**
  - Ensure proper access permissions for services like EMR and Glue.
- **(Optional) Set Up AWS Glue Catalog:**
  - For metadata management.

### **2. Collecting Sample Data**

- Obtain sample web server log files. You can generate logs or use publicly available datasets.
- **Example Log Entry:**
  ```
  127.0.0.1 - - [10/Oct/2023:13:55:36 -0700] "GET /index.html HTTP/1.1" 200 1024
  ```

### **3. Data Ingestion**

- Upload the log files to the S3 bucket under a folder like `s3://your-bucket/logs/raw/`.

### **4. Data Processing with PySpark**

#### **a. Setting Up an EMR Cluster**

- Launch an EMR cluster with Spark installed.
- Configure the cluster with the necessary bootstrap actions and security groups.

#### **b. Writing the PySpark Script**

- **Import Necessary Libraries:**

  ```python
  from pyspark.sql import SparkSession
  import re
  ```

- **Initialize Spark Session:**

  ```python
  spark = SparkSession.builder.appName("WebLogAnalysis").getOrCreate()
  ```

- **Read Raw Data from S3:**

  ```python
  logs_rdd = spark.sparkContext.textFile("s3://your-bucket/logs/raw/*")
  ```

- **Define Regular Expression Pattern:**

  ```python
  log_pattern = re.compile(
      r'^(?P<ip>\S+) \S+ \S+ \[(?P<date>.*?)\] "(?P<request>.*?)" (?P<status>\d{3}) (?P<size>\S+)'
  )
  ```

- **Parse Log Entries:**

  ```python
  def parse_log_line(log_line):
      match = log_pattern.match(log_line)
      if match:
          return match.groupdict()
      else:
          return None

  logs_parsed = logs_rdd.map(parse_log_line).filter(lambda x: x is not None)
  ```

- **Convert to DataFrame:**

  ```python
  logs_df = logs_parsed.toDF()
  ```

- **Data Cleaning and Transformation:**

  - Convert data types as needed.
  - Extract additional fields from the request column if necessary.

- **Example:**

  ```python
  from pyspark.sql.functions import split, col

  logs_df = logs_df.withColumn('method', split(col('request'), ' ').getItem(0)) \
                   .withColumn('endpoint', split(col('request'), ' ').getItem(1)) \
                   .withColumn('protocol', split(col('request'), ' ').getItem(2))
  ```

#### **c. Data Storage**

- **Write Processed Data to S3 in Parquet Format:**

  ```python
  logs_df.write.parquet("s3://your-bucket/logs/processed/", mode='overwrite')
  ```

### **5. Data Analysis with SparkSQL**

- **Register DataFrame as Temporary View:**

  ```python
  logs_df.createOrReplaceTempView("logs")
  ```

- **Example Queries:**

  - **Top 10 Requested Endpoints:**

    ```python
    top_endpoints = spark.sql("""
      SELECT endpoint, COUNT(*) as request_count
      FROM logs
      GROUP BY endpoint
      ORDER BY request_count DESC
      LIMIT 10
    """)
    top_endpoints.show()
    ```

  - **Status Code Distribution:**

    ```python
    status_distribution = spark.sql("""
      SELECT status, COUNT(*) as count
      FROM logs
      GROUP BY status
      ORDER BY count DESC
    """)
    status_distribution.show()
    ```

- **Save Query Results:**

  ```python
  top_endpoints.write.csv("s3://your-bucket/logs/output/top_endpoints.csv", mode='overwrite')
  ```

### **6. Automation with AWS Lambda and AWS Glue (Optional)**

- **Set Up AWS Glue Job:**
  - Create a Glue job to run the PySpark script on a schedule.
- **Set Up AWS Lambda Function:**
  - Trigger processing when new log files are uploaded to S3.

### **7. Visualization (Optional)**

- **Using Jupyter Notebooks on EMR:**

  - Install Jupyter on EMR and visualize data using Python libraries like Matplotlib or Seaborn.

- **Using AWS QuickSight:**

  - Connect QuickSight to the processed data in S3 and create dashboards.

---

## **Project Documentation**

- **README.md:**
  - Provide an overview, setup instructions, and explanations of each component.
- **Code Organization:**
  - Organize scripts into folders (e.g., `scripts/`, `notebooks/`).
- **Comments and Docstrings:**
  - Ensure your code is well-commented for clarity.

---

## **Best Practices**

- **Use Version Control:**
  - Keep your code in a Git repository with meaningful commit messages.
- **Handle Exceptions:**
  - Include error handling in your scripts.
- **Security:**
  - Do not hard-code AWS credentials; use IAM roles.
- **Optimization:**
  - Consider partitioning data for better performance.
- **Cleanup Resources:**
  - Remember to terminate EMR clusters when not in use to avoid unnecessary costs.

---

## **Demonstrating Skills**

- **Regular Expressions:**
  - Show how you used regex to parse unstructured log data.
- **SQL and SparkSQL:**
  - Include complex queries and explain their purpose.
- **Python and PySpark:**
  - Write efficient PySpark code and use DataFrame APIs.
- **Data Engineering Concepts:**
  - Illustrate your understanding of ETL processes and data pipelines.
- **Big Data Handling:**
  - Emphasize processing large datasets efficiently with Spark.

---

## **Additional Enhancements**

- **Implement Unit Tests:**
  - Use `unittest` or `pytest` frameworks to test your functions.
- **Continuous Integration:**
  - Set up a CI/CD pipeline with tools like Jenkins or GitHub Actions.
- **Containerization:**
  - Package your application using Docker for portability.
- **Streaming Data:**
  - Incorporate AWS Kinesis or Spark Streaming for real-time data processing.
