Mini-Tutorial: Connecting JupyterLab to PySpark
Goal: To set up a professional, stable, and interactive development environment for PySpark using JupyterLab, connected to our existing data lakehouse.

1. Installation
This section covers the one-time installation of the Python libraries and helper packages needed.

Activate your Conda environment (if you use one):

conda activate watcher-env

Install JupyterLab and findspark: findspark is a crucial helper library that ensures Jupyter connects to the correct Spark installation.

pip install jupyterlab findspark

Uninstall any conflicting PySpark library: This is the most important step to prevent version conflicts. We will force our environment to use the pyspark that comes with our main Spark engine.

pip uninstall pyspark

(It's okay if this says "not installed". The goal is to ensure it's not present in your Python environment.)

(Optional) Download Native Hadoop Libraries: This step is optional but recommended to fix the WARN NativeCodeLoader warning for a cleaner startup.

# Navigate to your main project directory
cd ~/minio

# Download and extract the matching Hadoop version
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xzf hadoop-3.3.6.tar.gz

2. The Startup Scripts
We use two scripts to reliably start our environment.

2.1. Backend Script (start_lakehouse.sh)
This script (located in ~/minio) starts MinIO and PostgreSQL. It remains unchanged.

2.2. Jupyter Launch Script (start_jupyter.sh)
This script prepares the environment variables and starts the JupyterLab server. It is critical that this script correctly points to both your Spark and Hadoop installations to avoid warnings and errors.

File Name: start_jupyter.sh

Location: ~/minio/

Final Correct Content:

#!/bin/bash

# Define the location of your Spark installation
export SPARK_HOME="$HOME/minio/spark-3.5.1-bin-hadoop3"

# Define the location of your Hadoop installation to fix the native library warning
export HADOOP_HOME="$HOME/minio/hadoop-3.3.6"

# Add Hadoop's native libraries to the system's library path
export LD_LIBRARY_PATH="$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH"

# --- Launch JupyterLab ---
echo "Starting JupyterLab..."
jupyter lab --ip=0.0.0.0 --notebook-dir=$HOME/minio

3. The Jupyter Notebook Connection Code
This is the block of code you will run in the first cell of your notebook every time you start or restart the kernel.

# 1. First, find Spark and add it to the environment. This MUST be run before importing pyspark.
import findspark
findspark.init('/home/ubuntu/minio/spark-3.5.1-bin-hadoop3')

# 2. Now you can import pyspark successfully
from pyspark.sql import SparkSession

# 3. Build the SparkSession with the full, correct configuration
spark = SparkSession.builder \
    .appName("IcebergJupyter") \
    .config("spark.jars.packages", "org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.2,org.postgresql:postgresql:42.7.3,software.amazon.awssdk:bundle:2.25.30,org.apache.hadoop:hadoop-aws:3.3.4") \
    .config("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.local", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.local.type", "jdbc") \
    .config("spark.sql.catalog.local.uri", "jdbc:postgresql://localhost:5432/iceberg_catalog") \
    .config("spark.sql.catalog.local.jdbc.user", "iceberg") \
    .config("spark.sql.catalog.local.jdbc.password", "iceberg") \
    .config("spark.sql.catalog.local.jdbc.schema-version", "V1") \
    .config("spark.sql.catalog.local.warehouse", "s3a://datalake/warehouse") \
    .config("spark.hadoop.fs.s3a.endpoint", "http://1.0.0.1:9000") \
    .config("spark.hadoop.fs.s3a.access.key", "spark-user") \
    .config("spark.hadoop.fs.s3a.secret.key", "spark-password-123") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .config("spark.hadoop.fs.s3a.connection.ssl.enabled", "false") \
    .config("spark.hadoop.fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem") \
    .getOrCreate()

print("SparkSession created successfully!")

4. Troubleshooting Summary
We encountered several common but tricky issues. Here is a summary of the problems and their solutions.

Problem 1: TypeError: 'JavaPackage' object is not callable (sometimes appearing after a 5-30 second delay).

Reason: This was the main issue. It was caused by a version conflict between the pyspark library installed in the Conda environment and the main Spark engine we downloaded. Our initial attempts to fix it created a race condition.

Solution: The definitive fix was to uninstall pyspark from the Conda environment (pip uninstall pyspark) and then use the findspark library in the notebook to reliably connect to the main Spark engine.

Problem 2: ModuleNotFoundError: No module named 'pyspark'.

Reason: This happened after uninstalling pyspark. The notebook code was trying to import pyspark before findspark.init() had a chance to tell Python where to find it.

Solution: Correct the order of the code. The findspark.init() command must be the first line of code executed in the session.

Problem 3: Exception: Unable to find py4j...

Reason: A simple typo in the path provided to findspark.init(). We had spark-3.5-1... instead of the correct spark-3.5.1....

Solution: Correct the typo in the path to the Spark home directory.

Problem 4: Py4JJavaError when running a spark.sql() command.

Reason: A typo in the PostgreSQL connection string. The port was incorrectly set to 5232 instead of the correct 5432.

Solution: Correct the port number in the .config("spark.sql.catalog.local.uri", ...) line.