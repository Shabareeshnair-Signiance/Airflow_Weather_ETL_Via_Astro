# Weather ETL Pipeline — Apache Airflow & Astro CLI

A small end-to-end ETL (Extract, Transform, Load) pipeline built with **Apache Airflow**, orchestrated locally using **Astro CLI**, that fetches live weather data for Kerala from the **Open-Meteo API** and stores it in a **PostgreSQL** database.

This project was built as a hands-on demo to learn Airflow DAG development, Astro CLI local environment setup, and Docker-based service orchestration.

---

## Overview

The pipeline runs three tasks in sequence:

1. **Extract** — Calls the Open-Meteo API to fetch current weather data (temperature, windspeed, wind direction, weather code) for a given latitude/longitude.
2. **Transform** — Cleans and structures the raw API response into a flat format ready for storage.
3. **Load** — Inserts the transformed data into a PostgreSQL table.

```
Open-Meteo API → extract_weather_data → transform_weather_data → load_weather_data → PostgreSQL
```

---

## Tech Stack

- **Apache Airflow** (Astro Runtime, Airflow 3.x)
- **Astro CLI** — local Airflow environment management
- **Docker / Docker Compose** — containerization
- **PostgreSQL** — data storage
- **Open-Meteo API** — free weather data source (no API key required)
- **Python** (TaskFlow API, HTTP Hook, Postgres Hook)

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [Astro CLI](https://www.astronomer.io/docs/astro/cli/install-cli) installed
- Windows users: Microsoft Hyper-V enabled

---

## Project Structure

```
ETLWeather/
├── dags/
│   └── etlweather.py              # Main DAG: extract, transform, load
├── docker-compose.override.yml    # Dedicated Postgres container for pipeline data
├── requirements.txt               # Airflow provider packages
├── airflow_settings.yaml          # Local Airflow connections (optional, for auto-setup)
├── Dockerfile                     # Astro Runtime base image
└── README.md
```

---

## Setup & Run Locally

**1. Clone the repository**
```bash
git clone <your-repo-url>
cd ETLWeather
```

**2. Start the local Airflow environment**
```bash
astro dev start
```
This builds the project image and spins up the Airflow containers (scheduler, triggerer, dag-processor, api-server) along with a dedicated Postgres container defined in `docker-compose.override.yml`.

**3. Open the Airflow UI**

Go to [http://localhost:8080](http://localhost:8080)

**4. Set up connections**

Under **Admin → Connections**, add:

| Connection ID | Type | Host | Login | Password | Port | Database |
|---|---|---|---|---|---|---|
| `open_meteo_api` | HTTP | `https://api.open-meteo.com` | — | — | — | — |
| `postgres_weather` | Postgres | `postgres_weather` | `myuser` | `1234` | `5432` | `weather` |

> Tip: These can also be pre-defined in `airflow_settings.yaml` so they're created automatically on every `astro dev start`.

**5. Trigger the DAG**

In the Airflow UI, find `weather_etl_pipeline` and click **Trigger**.

---

## Verifying the Output

Connect directly to the Postgres container to check the stored data:

```bash
docker exec -it postgres_weather_db psql -U myuser -d weather
```

```sql
SELECT * FROM weather_data;
```

**Sample output:**

| latitude | longitude | temperature | windspeed | winddirection | weathercode | timestamp |
|---|---|---|---|---|---|---|
| 10.8505 | 76.2711 | 25.9 | 14.5 | 263 | 96 | 2026-07-03 08:53:39 |

---

## Key Learnings

- Airflow Connections are configured separately from DAG code (via UI or `airflow_settings.yaml`), not hardcoded.
- Astro CLI manages its own internal Docker Compose setup; additional services must be added via `docker-compose.override.yml`.
- New Docker services must be explicitly attached to Astro's internal network to be reachable by other containers via service name.
- The official Postgres Docker image only applies `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` on the **first** initialization of a fresh volume — not on subsequent restarts.
- Astro Runtime 3.0+ no longer bundles the Postgres/HTTP providers by default; they must be added explicitly to `requirements.txt`.

---

## Author

Shabareesh Nair
