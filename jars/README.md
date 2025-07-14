# Pre-built JAR Files

This directory contains pre-built JAR files for the SQL Server Spark Connector, ready for download and deployment to Databricks clusters or other Spark environments.

## Available Files

- `spark-mssql-connector-1.4.0.jar` (84KB) - Main connector JAR containing the core functionality
- `spark-mssql-connector-1.4.0-javadoc.jar` (96KB) - Documentation JAR containing API documentation

## Usage

1. Download the main JAR file `spark-mssql-connector-1.4.0.jar` 
2. Upload it to your Databricks cluster or Spark environment
3. Use the connector in your Spark applications

## Build Information

These JARs were built with:
- Maven 3.9.10
- Java 17
- Scala 2.13.16 (default profile spark40)
- Spark 4.0.0

For the latest version or to build from source, see the main README.md in the repository root.