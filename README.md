
# **Deploy ETLT Infrastructure with Airbyte, GCS, BigQuery, Dbt and Airflow**
[![Airbyte](https://img.shields.io/badge/-Airbyte-2962FF?style=flat&logo=airbyte&logoColor=white)](https://airbyte.com/)
[![Google Cloud Storage](https://img.shields.io/badge/-GCS-4285F4?style=flat&logo=google-cloud&logoColor=white)](https://cloud.google.com/storage)
[![BigQuery](https://img.shields.io/badge/-BigQuery-34A853?style=flat&logo=google-bigquery&logoColor=white)](https://cloud.google.com/bigquery)
[![dbt](https://img.shields.io/badge/-dbt-FF694B?style=flat&logo=dbt&logoColor=white)](https://www.getdbt.com/)
[![Airflow](https://img.shields.io/badge/-Airflow-017CEE?style=flat&logo=apache-airflow&logoColor=white)](https://airflow.apache.org/)
[![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![VisualCrossing](https://img.shields.io/badge/-VisualCrossing-FFB300?style=flat&logo=visualcrossing&logoColor=white)](https://www.visualcrossing.com/)
[![Opendatasoft](https://img.shields.io/badge/-Opendatasoft-00B6E3?style=flat&logo=opendatasoft&logoColor=white)](https://www.opendatasoft.com/)




This project sets up Dokcer container to perform data extraction with Airbyte. That allows us to try new airbyte data extraction usecases.
It sets up an analytics architecture using Docker containers for data extraction, storage, orchestration, and transformation.

- **Airbyte (Extract):** Retrieves data from various sources and loads it into Google Cloud Storage (GCS).
- **Python Transformation:** Cleans and enriches files stored in GCS, optimizing the storage system before further processing. This step also transforms data formats from `.json` to `.parquet` for efficient storage and querying.
- **BigQuery (Load):** Loads the transformed data from GCS into BigQuery for scalable storage and querying.
- **dbt (Transform):** Executes SQL models in BigQuery to further transform and prepare the data for analysis.
- **Airflow (Orchestrate):** Manages and automates the workflow across all components.

Below we can see the flowchart of into the architecture of ELT/ETLT created:

![flowchart](assets/weather_etlt_diagram.svg)

---

## Local Installation

### 1. **Airbyte Installation**

Airbyte is installed locally via Docker in this setup. For production or Kubernetes environments, refer to the [Airbyte OSS installation guide](https://docs.airbyte.com/platform/deploying-airbyte). Example Helm installation:

```bash
helm install \
  airbyte \
  airbyte/airbyte \
  --namespace airbyte \
  --values ./values.yaml
```

We might also use this fastest way to install a QuickStart Aribyte by following different steps explained on [Airbyte website](https://docs.airbyte.com/platform/using-airbyte/getting-started/oss-quickstart).

Once Airbyte is installed, the first step is to connect it to your weather API (e.g., VisualCrossing). You can use Airbyte's API connector builder for this purpose. Review the [`custom_api_builder.yaml`](airbyte/custom_api_builder.yaml) file for connection details—this file outlines the configuration needed to connect to the VisualCrossing API, including authentication and endpoint setup. You can recreate the connector using Airbyte's no-code interface or import this YAML file.

To configure the connector, you will need:
- **API Key** from your VisualCrossing account.
- **Base URL**: `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timelinemulti` (supports up to five locations per request for free accounts). See [VisualCrossing documentation](https://www.visualcrossing.com/resources/documentation/weather-api/using-the-timeline-weather-api-with-multiple-locations-in-the-same-request/) for details.
- **Locations**: Specify locations in the format `London,UK|Paris,France|Tokyo,Japan|Cape Town,South Africa`.

For guidance on building custom API connectors, refer to the [Airbyte Connector Builder tutorial](https://docs.airbyte.com/platform/connector-development/connector-builder-ui/tutorial).

In this project, multiple sources with the same connector to the weather API are created—each handling a chunk of locations (to cover all 99 departments (96 metropolitan and 3 DOM) of  France)—with ``Google Cloud Storage`` as the destination.

Requirements
- Allow connections from Airbyte server to your GCS .
- An GCP bucket with credentials (for the COPY strategy).

If you encounter issues, consult the [Airbyte GCS destination documentation](https://docs.airbyte.com/integrations/sources/gcs) or reach out to the Airbyte community. You might find how to connect to GCS.

### 2. **Clone the repository**:
  ```bash
  git clone https://github.com/donat-konan33/EtltAirbyteGcsBigQueryDbtAirflow.git
  cd EtltAirbyteGcsBigQueryDbtAirflow
  ```

### 3. **Start the infrastructure**:
  ```bash
  docker compose up -d
  ```

### 4. **Access web interfaces**:
  - Airbyte: [http://localhost:8000](http://localhost:8000)
  - Airflow: [http://localhost:8080](http://localhost:8080)

You may also use [`DBeaver to overwiew`](https://stackoverflow.com/questions/70424637/connect-docker-postgres-from-outside-dbeaver) your Airflow metadata.

### 5. **Connect to BigQuery** using your preferred SQL client or the Google Cloud Console to view and validate dbt transformations.

---

## Notes

This project is under development.
Refer to each service's documentation for more details.
It is important to create a local `.env` file with the same contents as `.env.example`.

If you have questions or need clarification about any part of this setup, feel free to reach out for assistance.

## Environment Variables

Here are variables you need for this project by referring to [`.env.example`](.env.example):

| Variable                   | Description                                                           |
|----------------------------|-----------------------------------------------------------------------|
| **PROJECT_ID**             | Google Cloud project ID                                               |
| **GCP_SERVICE_ACCOUNT_KEY**      | Contents of your Google Cloud service account key (JSON)        |
| **GCP_SERVICE_ACCOUNT_KEY_PATH** | Path to your Google Cloud service account key file              |
| **PATH_TO_SERVICE_ACCOUNT_DIRECTORY** | Directory containing your service account key file         |
| **AIRBYTE_URL**            | URL for Airbyte instance                                              |
| **AIRBYTE_USER**           | Username for Airbyte                                                  |
| **AIRBYTE_PASSWORD**       | Password for Airbyte user                                             |
| **HOST**                   | Host address for Airbyte server                                       |
| **AIRFLOW_ADMIN_EMAIL**    | Email address for Airflow admin user                                  |
| **AIRFLOW_ADMIN_USERNAME** | Username for Airflow admin user                                       |
| **AIRFLOW_ADMIN_PASSWORD** | Password for Airflow admin user                                       |
| **AIRFLOW_ADMIN_FIRST_NAME**| First name of Airflow admin user                                     |
| **AIRFLOW_ADMIN_LAST_NAME** | Last name of Airflow admin user                                      |
| **LAKE_BUCKET**            | Name of your Google Cloud Storage bucket assigned as Airflow variable |

---

**Important**
``Streamlit`` source code for BigQuery data visualization can be found by following this [link](https://github.com/donat-konan33/BigQueryStreamlitAnalyticsDashboard).
