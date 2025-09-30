# HealthCare_GCP

**Agenda**

This project focuses on building scalable data lake in GCP for Revenue Cycle Management in the healthcare domain.


The primary objective is to centralize, clean, and transform data from multiple sources, enabling healthcare providers and insurance companies to streamline:
    Billing
    Claims processing
    Patient History
    Revenue tracking









**Project Architecture**


1. Data Sources: 
    - All the data from different sources that come from upstream (software engineers).  
    - These sources are the databases
2. Landing:
    - The data is ingested from the databases to the Google Cloud Storage.
3. Medillion Architecture:
    - Bronze:
        - The files are moved from the landing layer to the bronze layer.
        - This bronze layer is build on the BigQuery
    - Silver:
        - Slowly Changing Dimension (SCDs)
        - Common Data Model (CDM)
    - Gold:
        - Data Modeling
        - Visualization
4. Orchestration:
    - Schedule, orchestrate and manage pipelines.


![alt text](image.png)



<br><br><br><br>




Services:
- GCS: Store Raw and prcessed data
- BigQuery: Store, process, analyze the structured data
- Dataproc: Used for hadoop and Spark jobs (data processing, ingestion)
- Cloud COmposer: Airflow env (create and scedule DAGS)
- Cloud SQL: Store the transactional dbs
- GitHub: Store and version control
- CloudBuild: Automated deployment



<br><br><br><br>



Key:
1. Metadata-Driven Approch
2. SCD Type 2
3. Incremental Loading (Watermarking)
4. Common Data model (CDM)
5. Medallion Architecture
6. Logging and Monitoring
7. Error Handling
8. CI/CD
9. Automation
10. Optimization







<br><br><br><br>


Data sources:
1. EMR Data Sources - Cloud SQL Database
    - Stores data related to:
        - Patients table
        - Providers table
        - Departments table
        - Transactions table
        - Encounters table
    - There are two hospitals with sepearate dbs:
        - Hospital A -> hospital_a_db
        - Hospital B -> hospital_b_db

2. Claims Data Source
    - Comes from insurance companies
    - Provided as flat files
    - Stored in a designated folder in the data lake (landing zone) on a monthly bais.

3. CPT (Current Procedural Terminology) Codes:
    - A standardized system to describe medical, surgical, and diagnostic procedures.




<br><br><br><br><br><br><br>


Steps:
1. Create data sources:
    - hospital a database
    - hospital b database
    - claims csv files
    - cpt_codes csv files

Created two SQL instances in Cloud SQL in GCP with specific configurations and instance names as follows:
* hospital-a-mysql-db
* hospital-b-mysql-db  


    1. Create cloud SQL mysql instances ==> (hospital-a-mysql-db, hospital-b-mysql-db)
    2. All all network to access the instance
    3. Create database ==> hospital_a_db, hospital_b_db
    4. Create remote user
        - myuser
        - mypass
    5. Login to cloud sql studio
        - Run the scripts for creating the tab;es
        - Insert data (imported data)

    6. For claims and cpt_codes the file will be sent every month by upstream person in GCS.


2. Landing zone

Created a bucket in GCS for the landing layer:
* healthcare_bucket-28092025
    - data
        - has all the data that is ingetsed in the  cloud SQL
    - landing
        - Claims and cpt_codes files
        - 
    - configs
        - Metadata driven approch
            - Full Load (when there is less data and no need to chenge frequently)
            - Incremental Load (changes frequently) 
    - temp



<br><br><br><br><br><br><br><br><br><br>

**Data Ingestion**

- Create dataproc cluster
- 1-ingestion ==> hospital-a-mysql (tables) ==> gcs_bucket/landing/hospital-a/
- 2-ingestion ==> hospital-b-mysql (tables) ==> gcs_bucket/landing/hospital-b/


hospital-a-mysql --> patients ---> full data ---> gcs_bucket/landing/hospital-a/patients

- audit table (for modified date)




<br><br><br><br><br><br><br><br><br><br><br>


**Bronze**

External Table:
- the data is in GCS and the table structure is in the BigQuery.
Managed Table: 
- the data is availble in the BigQuery

<br><br>

- gcs_bucket/landing/hospital-a/* ==> external tables ==> bigquery/bronze_dataset/*
- gcs_bucket/landing/hospital-b/* ==> external tables ==> bigquery/bronze_dataset/*
- gcs_bucket/landing/claims/*.csv ==> dataproc ==> bigquery/bronze_dataset/claims
- gcs_bucket/landing/cptcodes/*.csv ==> dataproc ==> bigquery/bronze_dataset/cpt_codes


<br><br>

**Silver**
- bigquery/bronze_dataset/* ===> Full Load + truncate
- bigquery/bronze_dataset/* ===> CDM. + SCD2 