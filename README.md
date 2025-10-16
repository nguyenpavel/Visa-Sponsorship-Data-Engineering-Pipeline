# Visa Sponsorship Data Pipeline (AWS)

An end-to-end **cloud-native data engineering pipeline** that automatically aggregates, enriches, and updates daily data on **UK companies licensed to sponsor Skilled Worker visas**.

The system is built entirely on **AWS serverless architecture**, integrating public APIs, web crawlers, and PostgreSQL for analysis-ready data — designed for scalability, automation, and minimal maintenance overhead.

---
## System Architecture



## Core Features

### 1. Automated ETL Pipeline
- Extracts daily visa sponsor data from the official GOV.UK CSV dataset  
- Fetches company metadata using **Companies House API** (Search + Profile endpoints)  
- Scrapes verified **websites** and **LinkedIn profiles** via Python crawlers (`BeautifulSoup`, `requests`, Bing search)  
- Enriches dataset with **SIC code mappings** for industry classification  

```python
import psycopg2

conn = psycopg2.connect(
    dbname="postgres",
    user="postgres",
    password=os.getenv("DB_PASSWORD"),
    host="rds-endpoint.amazonaws.com"
)

cursor = conn.cursor()
cursor.execute("SELECT company_name, visa_route FROM Visa_table LIMIT 5;")
print(cursor.fetchall())
````

---

### 2. Data Processing & Storage

* Centralised **PostgreSQL RDS** designed for JSON compatibility and scalable updates
* Dynamic tables (`Visa_table`, `CompanyHouse_table`, `URL_table`, etc.) managed by Lambda functions
* Unique company identifiers generated via custom SQL function for historical traceability

```sql
CREATE OR REPLACE FUNCTION generate_unique_id()
RETURNS TEXT AS $$
BEGIN
  RETURN CONCAT(UUID_GENERATE_V4(), '_', TO_CHAR(NOW(), 'YYYYMMDD'));
END;
$$ LANGUAGE plpgsql;
```

---

### 3. Orchestration & Automation

* **AWS Lambda** for ingestion, transformation, and update processes
* **EventBridge** scheduler triggers **Step Functions** daily for end-to-end orchestration
* Full **CI/CD pipeline** integrating:

  * GitHub (source control)
  * CodeBuild (build and test)
  * CodeDeploy → Lambda (auto-deployment)

```yaml
# buildspec.yml example
version: 0.2
phases:
  build:
    commands:
      - pip install -r requirements.txt -t .
      - zip -r function.zip .
artifacts:
  files:
    - function.zip
```

---

### 4. Monitoring & Cost Management

* **CloudWatch dashboards** for Lambda performance metrics and error logging
* **AWS Budgets** with automated alerts for cost control and anomaly detection

---

## Technical Stack

| Layer            | Technology                                                                                   |
| ---------------- | -------------------------------------------------------------------------------------------- |
| Cloud            | AWS (Lambda, RDS, S3, EventBridge, Step Functions, CodeBuild, CodePipeline, CloudWatch, ECR) |
| Language         | Python (`psycopg2`, `boto3`, `requests`, `BeautifulSoup`)                                    |
| Database         | PostgreSQL (RDS)                                                                             |
| Containerization | Docker                                                                                       |
| CI/CD            | GitHub → AWS CodeBuild / CodePipeline                                                        |
| Logging          | CloudWatch                                                                                   |

---

## Outcomes

* Fully serverless and auto-updating **PostgreSQL database** covering 100,000+ visa-sponsoring companies
* Automated ingestion and transformation processes triggered daily with minimal manual intervention
* Analytical exports to **Parquet** for visualization or downstream ML integration
* Version-controlled CI/CD workflow enabling reproducible, modular updates

---

## Future Extensions

* Integrate **Amazon Textract** to extract structured data from PDF filings
* Add **Glassdoor API** for employee satisfaction and culture metrics
* Improve web crawler accuracy via domain ranking and keyword relevance scoring
* Build a **frontend dashboard** for interactive SQL-free data exploration

---

## Example SQL Query

```sql
SELECT 
  v.company_name,
  v.visa_route,
  c.city,
  c.status,
  s.industry,
  u.website_url,
  u.linkedin_url
FROM Visa_table v
JOIN CompanyHouse_table c ON v.unique_id = c.unique_id
JOIN SIC_table s ON c.sic_code = s.sic_code
JOIN URL_table u ON v.unique_id = u.unique_id;
```

---

## Repository Structure

```
├── lambdas/
│   ├── download_companynumber/
│   ├── download_companyhouse/
│   ├── webcrawler/
│   ├── create_unique_id/
│   └── update_urls/
├── sql/
│   ├── schema.sql
│   ├── functions.sql
│   └── queries/
├── buildspec.yml
├── Dockerfile
└── README.md
```
