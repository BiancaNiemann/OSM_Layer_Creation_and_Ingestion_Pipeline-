# 🧱 OSM Layer Creation & Ingestion Pipeline  
### Internship Project — Webeet.io (2 Months)

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![PostGIS](https://img.shields.io/badge/PostGIS-2E8B57?style=for-the-badge)
![OSMnx](https://img.shields.io/badge/OSMnx-000000?style=for-the-badge)
![GeoPandas](https://img.shields.io/badge/GeoPandas-139C5A?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 🎯 Project Overview
This project automates the **creation and population of Point of Interest (POI) layer tables** using data from **OpenStreetMap (OSM)**.  
Each layer is dynamically configured via database-driven metadata, allowing new POI layers to be added **without changing pipeline code**.

The pipeline handles schema creation, OSM data extraction, geospatial enrichment, and database insertion for the city of **Berlin**.

---

## 🗂️ Layer Configuration Table

A central configuration table defines **how each POI layer is built and populated**.

### 📝 List of Layers Table
This table is updated whenever a new POI layer is introduced.

**Stored metadata includes:**
- Unique layer ID  
- Table name  
- `osm_tags` (used to query the OSM API)  
- Columns to extract from OSM  
- Extra fields required for downstream unified tables  

All previously created OSM-based layers are registered here.

---

## 🛠️ Tools & Technologies

### 💻 Languages & Libraries
- Python  
- pandas  
- geopandas  
- osmnx  

### 🗄️ Databases & Spatial Tools
- PostgreSQL  
- PostGIS  

### 🐳 Environment
- Docker (containerized development & execution)

---

## 🔄 Pipeline Workflow

### 1️⃣ Import Libraries & Create Database Connection
- Loads required Python libraries
- Establishes a PostgreSQL connection via a shared helper function

---

### 2️⃣ Normalize OSM Tags (`normalize_osm_tags`)
**Purpose:**  
Convert raw and inconsistent OSM tag data into a clean format compatible with OSMnx.

**Features:**
- Accepts JSON strings, dictionaries, or lists of dictionaries  
- Merges multiple tag definitions into a single dictionary  
- Example:
  ```json
  [{ "office": "government" }, { "office": "administrative" }]
  ```
  ```json
  { "office": ["government", "administrative"] }
  ```
Outputs a normalized tag structure ready for filtering OSM data.

---

### 3️⃣ Fetch Layer Configurations

Reads layer metadata from `berlin_unified.berlin_layers_final_tables`.

**Extracts:**
- `table_name`
- `osm_tags`
- `columns`

Each row is:
- Stored as a Python dictionary
- Appended to a list of layer configurations

**Output:**  
A list of layer configs used to dynamically drive the pipeline.

---

### 4️⃣ Create Layer Tables

For each configured layer, the pipeline:

- Drops any existing table
- Creates a fresh table in `berlin_source_data.<table_name>`

**Applies column-specific rules:**
- `latitude`, `longitude` → `DECIMAL(9,6)`
- All other columns → `VARCHAR`

**Adds constraints when applicable:**
- `id` → `PRIMARY KEY`, `NOT NULL`
- `name` → `NOT NULL`
- `district_id` → `NOT NULL`

**Foreign keys:**
- `district_id` references `districts(district_id)`

Commits all changes and closes the database connection.

---

### 5️⃣ Load OSM Data into Each Table

For each layer, the pipeline:

- Fetches OSM data for Berlin using **OSMnx**

#### 🧹 Data Cleaning & Standardization
- Converts geometries to `Point`
- Reprojects data to WGS84 (`EPSG:4326`)
- Extracts latitude and longitude
- Fills missing names with `"unknown"`

#### 🌍 Spatial Enrichment
- Loads Berlin district boundaries from GeoJSON
- Performs spatial joins to assign districts
- Maps each district to a `district_id`
- Attaches neighborhood names from the database

#### 🧱 Schema Enforcement
- Ensures all expected columns exist
- Adds missing columns with `NULL`
- Converts geometry to WKT for database storage

#### 📥 Data Insertion
- Inserts records into `berlin_source_data.<table_name>`
- Uses `ON CONFLICT DO NOTHING` to prevent duplicates

---

### 6️⃣ Run All Layer Loads

- Iterates over every layer configuration
- Normalizes OSM tags
- Loads and inserts OSM data into the correct table

✅ Fully automates table creation and population for all registered POI layers.

---

## 📊 Loaded POI Layers (Examples)

Previously loaded layers include:
- banks
- galleries
- museums
- hospitals
- parks
- playgrounds
- supermarkets
- public_artworks
- theaters
- venues
- libraries
- pharmacies

> Some layers show minor count differences between OSM and the database due to spatial filtering and validation.

---

## 🚧 Pending / Custom Layers

Some layers require non-standard handling or additional logic:
- bike_lanes
- bus_stops
- tram_stops
- U-Bahn / S-Bahn
- food_markets
- schools & universities
- short_term_listings
- long_term_listings (currently copied manually)

---

## 🔮 Future Improvements
- Duplicate detection before insert
- Remove `contact:` and `addr:` prefixes from attributes
- Standardize non-OSM data sources
- Add automated logging & summary metrics
- Remove unwanted columns or columns with certain percent missing data

---

## 📌 Highlights
- Database-driven layer configuration (no hardcoded logic)
- Fully automated table creation and ingestion
- Robust geospatial enrichment using PostGIS
- Scalable design for adding new POI layers
- Production-ready structure aligned with data engineering best practices

