# DE-ZC-M2_Data_Orchestration

Data Orchestration project using Kestra for orchestrating NYC Taxi data pipelines on Google Cloud Platform.

## Overview

This project demonstrates a complete data orchestration solution using Kestra, featuring:
- Parent-child workflow patterns with nested loops
- GCP integration (Cloud Storage, BigQuery)
- Dynamic data pipeline execution for multiple taxi types (yellow, green)
- Yearly and monthly data aggregation

## Project Structure

### HW_Solution/

Contains the homework solution with production-ready workflows:

#### **gcp_taxi_parent.yaml**
Main orchestration flow that:
- Accepts inputs for taxi type (yellow/green), years, and months
- Uses nested `ForEach` loops to iterate through years and months
- Calls child flows for each year-month combination
- Executes yearly aggregation tasks after all months complete
- Calculates and logs total row counts per year using Python scripts

**Key Features:**
- Parameterized inputs with defaults (years 2019-2021, all 12 months)
- Conditional task execution with nested flow control
- Subflow pattern for reusable child workflows
- Yearly total calculation with recursive JSON parsing
- Error handling with `allowFailure` for robustness

#### **gcp_taxi_child.yaml**
Child flow that processes individual month-year combinations:
- Downloads compressed CSV data from NYC-TLC GitHub releases
- Extracts taxi trip data (yellow or green cab trips)
- Calculates row count and file size metrics
- Outputs structured data with metadata

**Key Features:**
- Dynamic file naming based on inputs
- Shell command execution for data extraction via curl and gunzip
- Output metrics (row count, file size in MB)
- Uses Kestra Key-Value store for GCP configuration
- Contains commented BigQuery integration code for future enhancements:
  - External table creation
  - Table merging with duplicate detection
  - Support for both yellow and green taxi schemas

**Inputs:**
- `taxi`: Taxi type (yellow or green)
- `year`: Data year (e.g., "2019")
- `month`: Data month (e.g., "01")

**Outputs:**
- `row_count`: Number of data rows processed
- `file_size`: File size in megabytes

#### **docker-compose.yml**
Infrastructure setup for running Kestra:

**Services:**
- **kestra_postgres**: PostgreSQL database for Kestra metadata and queue management
- **kestra**: Kestra v1.1 server running in standalone mode

**Configuration:**
- Basic authentication: `admin@kestra.io` / `Admin1234!`
- Web UI accessible at `http://localhost:8080`
- PostgreSQL driver for repository and queue persistence
- Docker socket mounted for task execution
- Support for `.env_encoded` file for GCP service account credentials

#### **service-account.json**
GCP service account credentials (not included for security - add your own)

## Setup & Execution

### Prerequisites
- Docker and Docker Compose
- GCP project with credentials
- NYC-TLC dataset access

### Installation

1. Clone the repository:
   ```bash
   cd HW_Solution
   ```

2. Create `.env_encoded` file with your GCP service account:
   ```bash
   echo "GCP_SERVICE_ACCOUNT=$(cat service-account.json)" > .env_encoded
   ```

3. Start Kestra:
   ```bash
   docker-compose up -d
   ```

4. Access Kestra UI at `http://localhost:8080`

### Creating Kestra Key-Value Store

Before running workflows, configure GCP settings in Kestra KV store:
- `GCP_PROJECT_ID`: Your GCP project ID
- `GCP_BUCKET_NAME`: Cloud Storage bucket name
- `GCP_DATASET`: BigQuery dataset name
- `GCP_LOCATION`: GCP region (e.g., "US")
- `GCP_SERVICE_ACCOUNT`: Service account JSON (stored as secret)

### Running Workflows

1. **Parent Workflow**: Trigger `gcp_taxi_parent` from the Kestra UI
2. **Parameters**: Select taxi type, years, and months
3. **Monitoring**: View execution logs and task metrics in real-time
4. **Results**: Check yearly aggregations and file processing statistics

## Workflow Logic

### Parent-Child Pattern

```
gcp_taxi_parent
├── For each Year
│   ├── For each Month
│   │   └── call gcp_taxi_child (Subflow)
│   ├── Calculate Yearly Total
│   └── Log Yearly Summary
```

### Data Flow

1. **Extract**: Download compressed taxi data from GitHub
2. **Parse**: Extract CSV and calculate metrics
3. **Store**: Output data ready for GCS upload (commented out)
4. **Transform**: BigQuery table creation and merging (commented out)
5. **Aggregate**: Calculate yearly totals from child outputs

## Technologies Used

- **Kestra**: Workflow orchestration and scheduling
- **PostgreSQL**: Metadata and state management
- **GCP Services**:
  - Google Cloud Storage (GCS)
  - Google BigQuery
  - Service Account authentication
- **Docker Compose**: Infrastructure as code

## Extensibility

The commented BigQuery code in `gcp_taxi_child.yaml` provides templates for:
- Creating partitioned tables by pickup datetime
- External table integration from GCS
- Duplicate detection using MD5 hashing
- MERGE operations for incremental loads
- Both yellow and green cab schema definitions

Enable these sections to create a complete ELT pipeline with data quality checks.

## Notes

- Workflows use dynamic variable rendering with Kestra expressions
- Error handling includes `allowFailure` for resilience
- File cleanup with `PurgeCurrentExecutionFiles` after processing
- Scalable design supports adding more years/months or taxi types
