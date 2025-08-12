# Spotify ETL Pipeline | AWS Serverless Data Engineering with Snowflake

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Apache Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)
![AWS Lambda](https://img.shields.io/badge/AWS%20Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)

---

## Overview
This project implements a **serverless ETL pipeline** that extracts music metadata from the **Spotify API**, processes it using **AWS Glue with PySpark**, and loads it into **Snowflake** using **Snowpipe** for analytics and dashboarding in BI tools like Power BI or Tableau.

The pipeline simulates a **real-world, production-grade music analytics workflow**, demonstrating cloud-native ETL, automation, and integration between AWS and Snowflake.

---

## Motivation
The primary goal was to build a **scalable, fully automated ETL pipeline** leveraging AWS serverless services to ingest, transform, and deliver data into Snowflake for analytics.  

It showcases the **end-to-end thinking of a Data Engineer** — from API ingestion design, storage structuring, PySpark transformations, automated loading, to BI consumption.

---

## Data Source
- **Source API:** [Spotify Web API](https://developer.spotify.com/documentation/)  
- **Data Extracted:**  
  - **Songs**: Name, ID, duration, popularity, added date, linked album & artist IDs  
  - **Albums**: Name, ID, release date, total tracks, external URL  
  - **Artists**: Name, ID, external URL  
- **Format:** Raw JSON extracted from Spotify playlists.

---

## Key Features
- **S3 Two-Zone Structure**:  
  - **`raw_data/`**
    - `to_processed/`: New JSON files awaiting transformation  
    - `processed/`: Archived JSON files after transformation  
  - **`transformed_data/`**
    - `album_data/`: Transformed CSVs for albums  
    - `artist_data/`: Transformed CSVs for artists  
    - `songs_data/`: Transformed CSVs for songs  
- **Automated Extraction**:
  - AWS Lambda function to call Spotify API every minute (via CloudWatch trigger)
- **Serverless Transformation**:
  - AWS Glue PySpark job to clean, normalize, and structure data
  - Uses **GlueContext** and **DynamicFrames** for schema inference and Spark-based processing
- **Snowflake Auto-Loading**:
  - Snowflake **stages** and **file formats** configured for S3
  - **Snowpipe** continuously loads transformed CSVs into tables
- **Analytics Ready**:
  - BI tools (Tableau, Power BI, Looker Studio) can directly query Snowflake tables

---

## Architecture

**Workflow**:  
1. **Spotify API → AWS Lambda** (Extract)  
2. **S3 (`raw_data/to_processed/`)** stores raw JSON  
3. **AWS Glue PySpark Job** (Transform)  
4. **S3 (`transformed_data/`)** stores CSV outputs  
5. **Snowpipe** (Load) auto-ingests into Snowflake tables  
6. **BI Tools** query Snowflake for analysis

![Spotify AWS Snowflake Architecture](images/architecture.png)

---

## 🔄 Workflow Overview

### 1. Ingestion Layer (Raw Zone)
- **Trigger:** CloudWatch Event every minute
- **Process:**
  - Lambda function uses `spotipy` to fetch playlist metadata
  - Writes JSON to `s3://.../raw_data/to_processed/`

---

### 2. Transformation Layer (Processed Zone)
- **AWS Glue with PySpark**:
  - Initializes **SparkContext** & **GlueContext**
  - Reads JSON from `to_processed/`
  - Extracts and flattens nested fields into tabular format:
    - `songs_data/` → CSV
    - `album_data/` → CSV
    - `artist_data/` → CSV
  - Writes to `transformed_data/` in S3
  - Moves original JSON to `processed/` and deletes from `to_processed/`

---

### 3. Loading Layer (Snowflake)
- **Storage Integration** with S3
- **File Format**: CSV with header skip and null handling
- **Stages**:
  - `spotify_stage` points to `transformed_data/`
- **Tables**:
  - `tbl_album`
  - `tbl_artists`
  - `tbl_songs`
- **Snowpipes** auto-load new files into tables

---

### 4. Consumption Layer (BI & Analytics)
- Snowflake tables queried directly via:
  - **Power BI**
  - **Tableau**
  - **Looker Studio**
- Example analytics:
  - Top artists by popularity
  - Song trends over time
  - Album releases by year

---

## Future Enhancements
- Implement **incremental load** for updates only  
- Add **data quality checks** with Great Expectations  
- Build **real-time dashboard** with streaming ingestion  
- Integrate with **AWS Step Functions** for orchestration  
- Extend to more Spotify endpoints (e.g., genres, user stats)

---

## 📷 Screenshots
1. **S3 Folder Structure**
   - `raw_data/` with `to_processed/` & `processed/`
   - `transformed_data/` with CSV outputs
2. **AWS Lambda Console**
   - Function configuration & CloudWatch triggers
3. **AWS Glue Job Run**
   - Successful PySpark transformations
4. **Snowflake Web UI**
   - Loaded tables and Snowpipe status
5. **BI Dashboard**
   - Insights visualized from Snowflake

*(Add actual screenshots in `/images` folder)*

---

## 📊 Example SQL Queries in Snowflake

```sql
-- Most Popular Artists
SELECT a.artist_name, AVG(s.popularity) AS avg_popularity
FROM tbl_songs s
JOIN tbl_artists a ON a.artist_id = s.artist_id
GROUP BY a.artist_name
ORDER BY avg_popularity DESC
LIMIT 10;

-- Album Releases by Year
SELECT YEAR(release_date) AS release_year, COUNT(*) AS album_count
FROM tbl_album
GROUP BY release_year
ORDER BY release_year;
```

---

## Repository Structure
```
├── extraction/spotify_api_data_extract.py             # AWS Lambda extraction script
├── transformation/spotify_transformation.py           # AWS Glue PySpark job
├── sql/Spotify_snowflake.sql                          # Snowflake DDL, stage, and Snowpipe scripts
├── images/                                            # Architecture & pipeline screenshots
├── docs/                                              # Additional documentation
└── README.md                                          # Project documentation
```

---

## Deployment Guide

### Prerequisites
- AWS Account with S3, Lambda, Glue access
- Snowflake account with integration permissions
- Spotify Developer API credentials

### Steps
1. **Set Up S3 Buckets**:
   - Create `raw_data` and `transformed_data` folders with subfolders as described
2. **Deploy Lambda Function**:
   - Add Spotify credentials to environment variables
   - Configure CloudWatch schedule
3. **Configure AWS Glue Job**:
   - Upload PySpark transformation script
   - Set up job with IAM role for S3 access
4. **Configure Snowflake**:
   - Run `sql/Spotify_snowflake.sql` to create stages, tables, and Snowpipes
5. **Connect BI Tool** to Snowflake

---

## Conclusion
This project demonstrates the design and implementation of a **serverless, cloud-native ETL pipeline** integrating **AWS** and **Snowflake** for Spotify music analytics.  

It highlights:
- **API integration** (Spotify Web API)
- **Serverless automation** (AWS Lambda, Glue, CloudWatch)
- **Scalable storage** (S3 with structured zones)
- **Automated loading** (Snowpipe)
- **Analytics readiness** (BI tools)

This work reflects my ability to:
- Architect **end-to-end data pipelines**
- Automate workflows for continuous ingestion
- Bridge engineering with analytics for actionable insights
