---
tags:
  - aws
share: "true"
date: 2024-01-14
---

# Amazon Athena
## Amazon Athena is an interactive query service that makes it easy to analyze data directly in Amazon Simple Storage Service (Amazon S3) using standard SQL

## Buzzwords / when to use Athena?
* analysis of semi-structured data and structured data stored in Amazon S3 (e.g. CSV, JSON, Apache Parquet, Apache ORC - columnar data formats)
* ad-hoc queries using ANSI SQL without the need to aggregate or load data into Athena


## Integrations
* Amazon QuickSight for visualisation
* Amazon Glue Data Catalog which offers persistent metadata store for your data in Amazon S3, create tables and query data in Athena based on central metadata store and integrated with ETL and data discovery features of AWS Glue.
* S3 - direct queries over data in S3 without a need to format the data or manage infrastructure (e.g. web logs, WAF logs etc.)
