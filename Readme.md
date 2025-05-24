Netflix Data Engineering Project

🚀 Project Overview

This project demonstrates an end-to-end data engineering pipeline using Azure Data Factory (ADF), Azure Data Lake Storage Gen2 (ADLS), and Databricks (Delta Live Tables). The dataset used is Netflix content metadata sourced from a public GitHub repository. The pipeline is built in three layers: Bronze, Silver, and Gold, following the Medallion architecture.

🧱 Architecture & Tools

Azure Data Factory (ADF): Data ingestion and validation from GitHub

Azure Data Lake Storage Gen2 (ADLS): Centralized storage for all layers

Databricks: Auto Loader, Delta Live Tables (DLT), workflows, and transformations

Delta Lake: Format for scalable and ACID-compliant tables

🪪 Data Sources

Dataset: Netflix Titles on GitHub

Files: netflix_cast.csv, netflix_directors.csv, netflix_countries.csv, netflix_categories.csv, netflix_titles.csv

🛠️ Step-by-Step Implementation

🔸 1. Data Ingestion Using ADF

Used ADF pipelines to validate if files exist in GitHub.

Iterated over files using ForEach activity.

Copied files into ADLS Bronze layer:

/bronze/netflix_cast/

/bronze/netflix_directors/

/bronze/netflix_countries/

/bronze/netflix_categories/

Skipped netflix_titles.csv in ADF to demonstrate Databricks Auto Loader.

🔸 2. Bronze to Silver Using Databricks Workflows

Created separate workflows for each entity.

Used widgets to pass folder name as parameter.

Read data from Bronze, transformed/validated it, and wrote to Silver layer:

/silver/netflix_cast/

/silver/netflix_directors/

/silver/netflix_countries/

/silver/netflix_categories/

🔸 3. Databricks Auto Loader for Netflix Titles

Used Auto Loader to ingest netflix_titles.csv directly to Bronze.

Workflow to run this only on Sundays using dbutils.widgets.get("weekday") logic.

🧪 Gold Layer with Delta Live Tables (DLT)

✅ Quality Rules

looktables_rules = {
    "rule1": "show_id is NOT NULL"
}

📁 Gold Tables from Silver Layer (Lookup Tables)

@dlt.table(name='gold_netflix_directors')
@dlt.expect_all(looktables_rules)
...

@dlt.table(name='gold_netflix_countries')
@dlt.expect_all(looktables_rules)
...

@dlt.table(name='gold_netflix_category')
@dlt.expect_all(looktables_rules)
...

@dlt.table(name='gold_netflix_cast')
@dlt.expect_or_drop("rule1", "show_id IS NOT NULL")
...

📄 Gold Table for netflix_titles

@dlt.table
def gold_stg_netflixtitles():
    df = spark.readStream.format('delta').load("abfss://silver@.../netflix_titles")
    return df

@dlt.view
def gold_trns_netflixtitles():
    df = spark.readStream.table("LIVE.gold_stg_netflixtitles")
    return df.withColumn("newflag", lit(1))

@dlt.table
@dlt.expect_all_or_drop({"rule1": "newflag IS NOT NULL", "rule2": "show_id IS NOT NULL"})
def gold_netflix_titles():
    df = spark.readStream.table("LIVE.gold_trns_netflixtitles")
    return df

🧾 Folder Structure

netflix-data-engineering/
├── code/
│   ├── code_notebook/
│   └── netflix_project.dbc
├── dataset/
│   ├── netflix_cast.csv
│   ├── netflix_category.csv
│   ├── netflix_countries.csv
│   ├── netflix_directors.csv
│   └── netflix_titles.csv
├── Images/
│   ├── adf_workflow_to_bronze.jpeg
│   ├── bronze_to_silver_for4files.jpeg
│   ├── datalake_container.jpeg
│   ├── dlt_gold_layer.jpeg
│   ├── Project_Architecture.jpeg
│   └── workflow_of_netflix_titles.jpeg
├── README.md


✅ Features Implemented



📈 Future Improvements

Add Power BI or Tableau dashboard on Gold layer

Implement Slowly Changing Dimension (SCD) logic for titles

Add column-level data quality checks (null %, data type expectations)

📬 Contact

For questions or collaboration, reach out at [your-email@example.com]

Built with 💻 Databricks, ADF, and Delta Lake