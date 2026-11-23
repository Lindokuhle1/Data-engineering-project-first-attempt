# Global Energy Transition Data Pipeline
## Project Requirements and Technical Architecture

**Document type:** Project Technical Specification  
**Version:** 1.0  
**Status:** Foundation and infrastructure established; historical ingestion pending  
**Primary focus:** Global energy transition, with South Africa as the primary geographical focus  
**Initial data provider:** Ember Energy API  
**Deployment approach:** Local-first, cloud-ready  

---

# 1. Project Overview

## 1.1 Introduction

The Global Energy Transition Data Pipeline is a data engineering and analytics project designed to collect, store, process and analyse energy-sector data to understand the transition from fossil fuels to renewable energy.

The project will use historical energy datasets from Ember Energy to investigate changes in electricity generation, electricity demand, renewable energy adoption, installed capacity and power-sector carbon emissions.

South Africa will be the primary country of analysis, with international comparisons used to provide a broader understanding of global energy transition trends.

The project will initially operate locally using Python and PostgreSQL. Its architecture will be designed to support future deployment to Microsoft Azure without requiring significant changes to the application code.

## 1.2 Main Objective

Build an automated, scalable and maintainable data pipeline that collects historical energy data from external APIs, preserves the original source data, transforms it into analytical datasets and makes it accessible through dashboards and reports.

The project should demonstrate practical data engineering principles, including:

- API integration and data acquisition.
- Historical data backfilling.
- Raw data preservation.
- Data validation and quality assurance.
- ETL/ELT pipeline design.
- Relational database management.
- Analytical data modelling.
- Data visualisation and reporting.
- Infrastructure as Code.
- Automated testing and version control.

## 1.3 Analytical Questions

The project will investigate four primary questions:

1. Is South Africa becoming more dependent on renewable energy?
2. How does South Africa compare with other countries in renewable electricity generation?
3. Is renewable energy adoption associated with lower electricity-sector carbon emissions?
4. How is global electricity generation changing over time?

These questions will guide the selection of datasets, transformations, analytical metrics and dashboard components.

---

# 2. Project Scope

## 2.1 Geographical Scope

South Africa will be the primary geographical focus.

International comparisons will initially include:

- South Africa
- Germany
- China
- India
- United States
- Brazil
- World aggregate, where available

The final geographical selection will be validated against the actual coverage provided by the Ember API.

## 2.2 Energy Scope

The project will initially investigate two major areas.

**A. Electricity generation and energy sources**

- Renewable electricity generation.
- Fossil-fuel electricity generation.
- Nuclear electricity generation.
- Electricity generation by individual energy source.
- Electricity demand.
- Installed electricity generation capacity.

**B. Environmental impact**

- Power-sector carbon emissions.
- Carbon emissions intensity.
- Changes in emissions over time.
- Relationships between renewable electricity generation and emissions.

The project will focus on electricity-sector indicators. It will not initially attempt to measure the entire energy economy, including every transport, industrial and household energy activity.

## 2.3 Initial Data Source

**Ember Energy API**

Official documentation:

- https://ember-energy.org/data/api/
- https://api.ember-energy.org/v1/docs
- https://api.ember-energy.org/v1/openapi.json

Ember will be the initial and sole external data source.

Potential datasets include:

| Dataset | Purpose |
|---|---|
| Electricity generation | Analyse electricity production by energy source |
| Electricity demand | Understand electricity consumption trends |
| Power-sector emissions | Measure changes in electricity-sector emissions |
| Carbon intensity | Analyse emissions per unit of electricity generated |
| Installed capacity | Track changes in electricity generation capacity |

Actual endpoint availability, geographical coverage, date ranges and response structures must be verified against the API documentation.

---

# 3. Technology Stack

The project will use the following technologies.

| Technology | Responsibility |
|---|---|
| Python | Application and pipeline development |
| PostgreSQL | Primary database |
| Pandas | Data processing and analysis |
| SQLAlchemy | Database abstraction and SQL integration |
| Psycopg | PostgreSQL connectivity |
| Requests | HTTP API communication |
| Pytest | Automated testing |
| Streamlit | Interactive analytics dashboards |
| Docker | Application and database containerisation |
| Docker Compose | Local infrastructure orchestration |
| Terraform | Future cloud infrastructure provisioning |
| Git and GitHub | Version control and project management |

## 3.1 Development Approach

The project will be developed locally first.

The application must not depend on a specific hosting environment. Database connections, API credentials and environment-specific settings must be configured externally.

This will allow the same Python application to connect to:

- Local PostgreSQL during development.
- Containerised PostgreSQL through Docker.
- Azure Database for PostgreSQL in a future deployment.

---

# 4. System Architecture

## 4.1 High-Level Architecture

The pipeline will follow a layered data architecture.

**External source → Ingestion → Bronze → Silver → Gold → Analytics**

Each layer has a specific responsibility.

### Bronze Layer — Raw Data

Purpose: Preserve data exactly as received from the source.

Responsibilities:

- Retrieve API responses.
- Preserve the complete original response.
- Store individual records from the response.
- Capture request and ingestion metadata.
- Record failed requests and ingestion errors.
- Maintain a relationship between batches and their raw records.

No business transformations, cleaning, normalisation or aggregation should happen in Bronze.

### Silver Layer — Cleaned Data

Purpose: Produce reliable, consistent and structured datasets.

Responsibilities:

- Validate records.
- Handle missing or invalid values.
- Standardise data types.
- Normalise geographical and energy-source references where necessary.
- Validate dates and measurement values.
- Identify duplicates according to defined business rules.
- Track data-quality issues.

Silver transformations must not destroy the original Bronze records.

### Gold Layer — Analytical Data

Purpose: Prepare data for business intelligence and analytical applications.

Responsibilities:

- Create analytical tables and views.
- Calculate renewable electricity shares.
- Calculate fossil-fuel electricity shares.
- Analyse carbon intensity trends.
- Compare countries.
- Create time-series datasets.
- Support dashboard and reporting requirements.

Gold should contain business-ready data rather than raw API responses.

## 4.2 Logical Architecture

The logical architecture defines the responsibilities of each component.

1. API client: communicates with Ember.
2. Ingestion service: controls data collection.
3. Storage service: persists API batches and raw records.
4. Database: stores Bronze, Silver and Gold data.
5. Transformation service: processes Bronze data into Silver and Gold.
6. Analytics application: presents processed information through Streamlit.

## 4.3 Physical Architecture

The initial physical architecture will consist of two Docker containers.

**Container 1: Python application**

Responsibilities:

- Ember API client.
- Historical data collector.
- Database connectivity.
- Data validation and transformation.
- Automated tests.
- Future scheduling capabilities.

**Container 2: PostgreSQL database**

Responsibilities:

- Persistent data storage.
- Bronze, Silver and Gold schemas.
- Database constraints and indexes.
- Transaction management.

Docker Compose will manage both containers and their network connectivity.

Redis, Kafka, Airflow and other orchestration services are intentionally excluded from the initial implementation.

---

# 5. Database Architecture

## 5.1 Database

Database name: `energy_transition`

PostgreSQL will use separate schemas to organise the different data layers.

| Schema | Purpose |
|---|---|
| bronze | Original source data and ingestion metadata |
| silver | Validated and cleaned datasets |
| gold | Analytical datasets and reporting views |

## 5.2 Bronze Database Design

The Bronze schema has three tables.

### Table: bronze.api_batches

Stores metadata and the original response for each API request.

| Column | Type | Description |
|---|---|---|
| batch_id | UUID | Unique batch identifier |
| source | TEXT | Data provider |
| endpoint | TEXT | API endpoint |
| request_parameters | JSONB | Request parameters, excluding secrets |
| http_status | INTEGER | HTTP response status |
| request_timestamp | TIMESTAMPTZ | Time request was made |
| ingested_at | TIMESTAMPTZ | Time data was stored |
| response_body | TEXT | Original response text |
| response_json | JSONB | JSON representation of the response |
| record_count | INTEGER | Number of records received |

### Table: bronze.raw_records

Stores individual records extracted from the API response.

| Column | Type | Description |
|---|---|---|
| record_id | BIGINT | Unique record identifier |
| batch_id | UUID | Associated API batch |
| record_index | INTEGER | Original position in the response |
| raw_data | JSONB | Original individual record |
| ingested_at | TIMESTAMPTZ | Time record was stored |

The combination of `batch_id` and `record_index` identifies a record's original position within a response.

### Table: bronze.ingestion_errors

Stores information about failed ingestion operations.

| Column | Type | Description |
|---|---|---|
| error_id | BIGINT | Unique error identifier |
| batch_id | UUID | Associated batch, where available |
| endpoint | TEXT | API endpoint |
| http_status | INTEGER | HTTP status, where available |
| error_message | TEXT | Error description |
| occurred_at | TIMESTAMPTZ | Time error occurred |

## 5.3 Bronze Storage Rules

The following rules apply:

1. Preserve the complete original API response.
2. Store each object in the response's `data` array as an individual raw record.
3. Do not change the original field names or measurement values.
4. Do not remove duplicate measurements in Bronze.
5. Retain the relationship between batches and records.
6. Store failed ingestion attempts in the error table.
7. Never store API credentials in request metadata or response logs.

`response_body` preserves the original response text. `response_json` provides a queryable JSONB representation. JSONB does not preserve the original formatting or key order, which is why both representations are retained.

## 5.4 Current Database Status

The following foundation has already been established:

- PostgreSQL 15.18 installed locally.
- Database `energy_transition` created.
- Schema `bronze` created.
- Tables `api_batches`, `raw_records` and `ingestion_errors` created.
- Primary keys, foreign keys, indexes and constraints established.
- Initial database backup created.

The Bronze tables are currently empty. The next milestone is to populate them through the Ember API.

Silver and Gold schemas will be designed and implemented when their respective transformation requirements are established.

---

# 6. Data Ingestion Strategy

## 6.1 Historical Backfill First

The project will begin with historical data collection.

The pipeline will retrieve the available historical datasets from Ember before live or scheduled ingestion is introduced.

This approach allows the project to establish a reliable historical baseline and validate the ingestion process before adding automation.

## 6.2 Initial Pilot Dataset

The first endpoint to implement will be yearly carbon intensity.

Endpoint:

`GET https://api.ember-energy.org/v1/carbon-intensity/yearly`

The previously inspected response contained:

- 5,677 records.
- Coverage from 2000 to 2025.
- A `stats` section containing request-level information.
- A `data` array containing individual measurements.
- The metric `emissions_intensity_gco2_per_kwh`.

The actual response and available data should be checked again during implementation.

## 6.3 Ingestion Process

The ingestion process will follow these steps:

1. Load configuration and API credentials.
2. Build the API request.
3. Send the HTTP request.
4. Validate the HTTP response.
5. Capture the complete response body.
6. Parse the JSON response.
7. Extract the records from the `data` array.
8. Create a unique batch identifier.
9. Insert batch metadata and the original response into `bronze.api_batches`.
10. Insert each individual record into `bronze.raw_records`.
11. Commit the transaction.
12. Verify the number of records stored.

If ingestion fails, the pipeline should capture useful error information without exposing credentials.

The batch and associated raw records should be written within a database transaction to avoid partially stored batches.

## 6.4 API Response Handling

Known API behaviour observed during documentation review:

| HTTP status | Meaning |
|---|---|
| 200 | Successful request |
| 403 | Incorrect or unauthorised API key |
| 404 | Incorrect endpoint |

The implementation should also handle general network failures, malformed JSON and unexpected HTTP responses defensively.

The API documentation currently indicates no rate limiting. Nevertheless, the client should have configurable timeouts and controlled retries for transient failures.

## 6.5 Historical Backfill and Future Live Ingestion

Historical backfill and live ingestion will be treated as separate operational modes.

**Historical backfill**

- Collect historical datasets.
- Capture all available records.
- Preserve every API response.
- Track completed and failed batches.
- Verify coverage and record counts.

**Future scheduled ingestion**

- Collect newly available data.
- Identify the appropriate collection window.
- Track ingestion progress.
- Avoid unnecessary repeated API requests.
- Preserve every received batch.

Any deduplication or incremental-update logic must be implemented outside Bronze.

---

# 7. Infrastructure as Code

## 7.1 Local Infrastructure

Docker Compose will define the local environment.

It will manage:

- Python application container.
- PostgreSQL container.
- Persistent database volume.
- Internal application-to-database networking.
- Environment configuration.
- Database health checks.

PostgreSQL data must be stored in a named Docker volume to ensure it survives container restarts and recreation.

The application will connect to PostgreSQL through the Docker service hostname when running inside the Compose network.

## 7.2 Future Cloud Infrastructure

Terraform will be introduced when the project is ready for cloud deployment.

Microsoft Azure is the preferred future hosting platform.

Potential Azure infrastructure:

| Azure service | Responsibility |
|---|---|
| Azure Resource Group | Resource organisation |
| Azure Database for PostgreSQL | Managed database |
| Azure Functions | Scheduled or event-driven ingestion |
| Azure Key Vault | Secret management |
| Azure Monitor | Monitoring and logging |
| Azure Storage | Terraform remote state and supporting storage |
| Azure networking | Secure service connectivity |

These services are future architectural considerations, not part of the initial local deployment.

Terraform configurations will be separated into reusable modules and environment-specific configurations for development, staging and production.

Terraform state files and credentials must not be committed to Git.

---

# 8. Project Directory Structure

The proposed structure separates application code, infrastructure, tests, documentation and operational scripts.

```text
global-energy-transition-pipeline/
│
├── infrastructure/
│   ├── local/
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── app/
│   │   │   └── Dockerfile
│   │   └── postgres/
│   │
│   └── cloud/
│       └── terraform/
│           ├── modules/
│           │   ├── resource-group/
│           │   ├── database/
│           │   ├── networking/
│           │   └── monitoring/
│           │
│           └── environments/
│               ├── dev/
│               ├── staging/
│               └── production/
│
├── src/
│   └── energy_pipeline/
│       ├── ingestion/
│       ├── storage/
│       ├── transformations/
│       ├── analytics/
│       ├── config/
│       └── __init__.py
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── scripts/
│
├── docs/
│   ├── architecture.md
│   ├── infrastructure.md
│   ├── data_dictionary.md
│   └── ingestion_strategy.md
│
├── .env.example
├── .gitignore
├── pyproject.toml
└── README.md
```

The directory structure may evolve as implementation requirements become clearer. The application should remain modular, with clear separation between API communication, database persistence and transformation logic.

---

# 9. Security and Configuration

The project will follow basic security practices from the beginning.

- API keys must be stored in environment variables.
- Local secrets must be stored in `.env` files excluded from Git.
- `.env.example` must contain placeholders rather than real credentials.
- Database credentials must not be hardcoded.
- Database connections should use configuration rather than fixed connection strings.
- Logs must not expose API keys or passwords.
- The application container should run as a non-root user.
- Database backups should be kept separately from source code.

Expected configuration variables include:

```text
EMBER_API_KEY=
POSTGRES_DB=energy_transition
POSTGRES_USER=
POSTGRES_PASSWORD=
DATABASE_URL=
APP_ENV=development
LOG_LEVEL=INFO
```

The actual values will be supplied locally through environment configuration.

---

# 10. Testing and Data Quality

Testing will be introduced alongside implementation rather than postponed until the end.

## 10.1 Unit Testing

Test individual application components:

- API client behaviour.
- Response parsing.
- Configuration loading.
- Batch creation.
- Record extraction.
- Error handling.

## 10.2 Integration Testing

Test interactions between components:

- Python application and PostgreSQL.
- Successful API response storage.
- Raw record insertion.
- Batch-to-record relationships.
- Database transaction behaviour.
- Error persistence.

## 10.3 Data Quality Checks

The pipeline should eventually validate:

- Expected fields are present.
- Record counts match the source response.
- Date values are valid.
- Measurements have appropriate data types.
- Geographic identifiers are available where expected.
- No records are silently lost during ingestion.

Bronze validation should focus on acquisition integrity, not on imposing Silver-layer business rules on the original data.

---

# 11. Analytics and Dashboard Requirements

Streamlit will provide the initial user interface.

The dashboard will be developed after sufficient historical data has been ingested and validated.

Potential dashboard sections:

**South Africa Energy Transition**
- Renewable electricity share.
- Fossil-fuel electricity share.
- Electricity generation trends.
- Installed renewable capacity.
- Power-sector emissions.
- Carbon intensity over time.

**International Comparison**
- Renewable electricity share by country.
- Electricity generation by energy source.
- Carbon intensity comparisons.
- Country-level historical trends.

**Global Energy Trends**
- Global renewable electricity growth.
- Fossil-fuel generation trends.
- Changes in electricity demand.
- Global power-sector emissions.

**Data Pipeline Monitoring**
- Total API batches.
- Total raw records.
- Successful and failed ingestion operations.
- Last ingestion timestamp.
- Dataset coverage.
- Data-quality indicators.

The dashboard must distinguish between measured values, calculated metrics and derived indicators.

---

# 12. Development Roadmap

The project will be implemented incrementally.

| Phase | Description | Status |
|---|---|---|
| 1 | Project requirements and architecture | Established |
| 2 | Project directory structure | In progress |
| 3 | PostgreSQL database and Bronze schema | Established |
| 4 | Docker Compose infrastructure | Pending |
| 5 | Python application container | Pending |
| 6 | Ember API client | Pending |
| 7 | Historical Bronze ingestion | Next major milestone |
| 8 | Additional historical datasets | Pending |
| 9 | Silver transformations and validation | Pending |
| 10 | Gold analytical modelling | Pending |
| 11 | Automated testing and pipeline reliability | Ongoing |
| 12 | Streamlit dashboards | Pending |
| 13 | Scheduled ingestion | Pending |
| 14 | Terraform and Azure deployment | Future |

## Immediate Development Priorities

The next implementation activities are:

1. Complete the local Docker Compose infrastructure.
2. Configure the PostgreSQL container and persistent volume.
3. Configure the Python application container.
4. Configure local environment variables.
5. Implement the Ember API client.
6. Test the yearly carbon-intensity endpoint.
7. Store the first API response in `bronze.api_batches`.
8. Store the individual measurements in `bronze.raw_records`.
9. Verify that the stored record count matches the API response.
10. Expand the collector to the remaining selected datasets.

---

# 13. Engineering Principles

The project will follow these principles throughout development.

**Preserve source data:** Original API responses must remain available for future reprocessing and auditing.

**Separate responsibilities:** API communication, storage, transformations and analytics should not be tightly coupled.

**Make ingestion traceable:** Every stored measurement must be traceable to its source batch.

**Build incrementally:** Implement and validate one dataset before expanding to additional endpoints.

**Design for reliability:** Use transactions, error tracking, timeouts, tests and persistent storage.

**Keep infrastructure reproducible:** Local environments should be deployable using Docker Compose, with future cloud infrastructure managed through Terraform.

**Avoid unnecessary complexity:** Introduce additional infrastructure only when there is a clear operational or technical requirement.

**Maintain version control:** All application code, database migrations, infrastructure definitions and documentation should be tracked through Git.

---

# 14. Expected Final Deliverables

At completion, the project should provide:

1. A working historical energy-data ingestion pipeline.
2. A PostgreSQL database containing raw, cleaned and analytical datasets.
3. Reusable Python API integration components.
4. Automated tests covering critical pipeline operations.
5. Docker Compose infrastructure for local deployment.
6. Terraform configurations for future Azure deployment.
7. Data-quality and pipeline-monitoring capabilities.
8. Interactive Streamlit dashboards.
9. Technical documentation and database data dictionaries.
10. A GitHub repository demonstrating the complete data engineering workflow.

## Conclusion

The Global Energy Transition Data Pipeline will provide a structured way to collect and analyse historical electricity-sector data, with South Africa as its primary focus and international comparisons providing additional context.

The implementation will begin with reliable historical data acquisition and raw-data preservation. Cleaning, analytical modelling, visualisation and cloud deployment will follow once the ingestion foundation has been validated.

The central engineering objective is to build a pipeline that is reproducible, traceable, maintainable and capable of growing from a local development project into a production-oriented data platform.
