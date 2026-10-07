# GeoVision – AI-Powered Spatial Intelligence Platform

GeoVision is a modular spatial intelligence platform for automated site and urban-area assessment.

The platform accepts a land-lot reference, resolves its geographic location, performs multiple spatial analyses, generates analytical maps, and produces a consolidated PDF report through a FastAPI-based backend.

The system combines geospatial datasets, OpenStreetMap information, spatial network analysis, environmental modelling, and automated visualization to provide a structured view of a site's surrounding environment.

> Note: The current implementation is primarily based on geospatial algorithms, GIS data, network analysis, and mathematical models. The "AI-Powered" name represents the project's intelligent automation direction; no standalone machine-learning model is currently documented as part of the core analysis pipeline.

---

## Overview

A site's potential can depend on several surrounding factors, including accessibility, transportation, land use, infrastructure, visual surroundings, and environmental conditions.

GeoVision brings these factors into a single automated workflow.

```text
                         LOT / SITE INPUT
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Location Resolver   │
                     └──────────┬──────────┘
                                │
                                ▼
                       Geographic Location
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
          OSM Data          GIS Datasets      Spatial Data
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ▼
                       Spatial Analysis Engine
                                │
       ┌────────────┬───────────┼───────────┬────────────┐
       ▼            ▼           ▼           ▼            ▼
    Walking      Driving    Transport    Context       View
       │            │           │           │            │
       └────────────┴───────────┴───────────┴────────────┘
                                │
                                ▼
                       Noise Assessment
                                │
                                ▼
                    Maps + Automated PDF Report
```

---

# Key Capabilities

### Site Resolution
Convert a human-readable lot identifier into geographic coordinates suitable for spatial processing.

### Walking Accessibility
Measure pedestrian reachability and proximity to important amenities.

### Driving Connectivity
Analyse road-network accessibility using travel-time-based isochrones.

### Public Transport Analysis
Evaluate nearby bus and rail infrastructure and calculate a transport accessibility score.

### Urban Context & Zoning
Combine zoning, buildings, roads, green spaces, and amenities into a contextual spatial map.

### 360° View Analysis
Classify the visual character surrounding a site into directional sectors.

### Noise Assessment
Estimate road traffic noise across the surrounding area using a grid-based propagation model.

### Automated Reporting
Generate a multi-page PDF containing the complete site assessment.

---

# Technology Stack

## Backend

- Python
- FastAPI
- Uvicorn
- REST APIs

## Geospatial Processing

- GeoPandas
- Shapely
- PyProj
- OSMnx
- NetworkX
- Contextily

## Data Processing

- NumPy
- Pandas
- Scikit-learn

## Visualization & Reporting

- Matplotlib
- ReportLab

## Data Sources

- OpenStreetMap
- Government GIS services
- GeoPackage datasets

The documented requirements include FastAPI, Uvicorn, GeoPandas, OSMnx, Contextily, Shapely, PyProj, NetworkX, NumPy, Pandas, Matplotlib, Requests, Scikit-learn, and ReportLab.

---

# Project Architecture

```text
GeoVision/
│
├── app.py
├── requirements.txt
├── runtime.txt
├── render.yaml
│
├── data/
│   ├── BUILDINGS_FINAL.gpkg
│   └── ZONE_REDUCED.gpkg
│
├── images/
│   └── alkf-logo.png
│
├── static/
│   └── HK_MTR_logo.png
│
└── modules/
    ├── resolver.py
    ├── walking.py
    ├── driving.py
    ├── transport.py
    ├── context.py
    ├── view.py
    └── noise.py
```

Each analysis capability is separated into its own module, while `app.py` acts as the FastAPI application entry point and route layer.

---

# Spatial Analysis Modules

## 1. Location Resolver

The resolver converts a lot identifier such as:

```text
IL 1657
```

into geographic coordinates.

### Processing Flow

```text
Lot Identifier
      ↓
Government GIS Resolver
      ↓
Raw Spatial Coordinates
      ↓
EPSG:4326
      ↓
EPSG:3857
```

The documented implementation receives source coordinates in EPSG:2326, transforms them to EPSG:4326 for OpenStreetMap queries, and then to EPSG:3857 for internal spatial processing.

---

## 2. Walking Accessibility

The walking module evaluates pedestrian accessibility from the selected site.

### Workflow

```text
Site Centroid
     ↓
Pedestrian Network Extraction
     ↓
Nearest Network Node
     ↓
Shortest-Path Analysis
     ↓
Amenity Identification
     ↓
Accessibility Zones
     ↓
Map Generation
```

The implementation uses OSMnx to obtain a walking network, snaps the site to a nearby network node, calculates shortest paths, categorizes amenities, and generates 5-minute and 10-minute accessibility areas.

### Amenity Categories

- Food
- Healthcare
- Education
- Retail
- Recreation

Default parameters documented in the project include a 1,200 m graph radius and a walking speed of 1.39 m/s.

---

## 3. Driving Connectivity

The driving module evaluates road accessibility from the selected site.

### Analysis

- Build a driving network using OSMnx
- Assign travel-time weights to road segments
- Generate drive-time isochrones
- Calculate road-network betweenness centrality
- Visualize important road corridors

### Accessibility Levels

```text
5 minutes
10 minutes
20 minutes
```

The resulting visualization contains accessibility polygons together with a contextual road-network and centrality representation.

---

## 4. Public Transport Accessibility

The transport module evaluates the availability of public transportation around the site.

It considers:

- Bus stops
- Rail stations
- MTR infrastructure
- Transit-node density
- Route diversity

The documented transport score is calculated as:

```text
Transit Score =
0.5 × Normalized Node Density
+
0.5 × Normalized Route Diversity
```

Transit nodes are displayed with visual emphasis based on route information.

---

## 5. Context & Zoning

The context module provides a spatial overview of the site's surrounding urban environment.

### Included Layers

```text
Zoning
Building Footprints
Road Network
Green Spaces
Amenities
Building Density
```

Zoning is taken from the local zoning GeoPackage, while amenities and other geographic context can be derived from OpenStreetMap.

---

## 6. 360° View Classification

GeoVision divides the area surrounding a site into directional sectors to characterize its surrounding environment.

The documented default configuration uses:

```text
36 sectors
10° per sector
```

Each sector considers:

- Green-space proportion
- Water presence
- Building density
- Average building height

The features are normalized and combined to determine the dominant view category.

### View Categories

| Category | Interpretation |
|---|---|
| GREEN VIEW | Strong green-area presence |
| WATER VIEW | Dominant water features |
| CITY VIEW | Higher building density and height |
| OPEN VIEW | Lower building density and height |

The scoring logic documented by the project is:

```text
Green Score = Green Ratio

Water Score = Water Ratio

City Score = Height × Density

Open Score = (1 - Density) × (1 - Height)
```

Each sector receives the classification with the highest score.

---

## 7. Road Traffic Noise Analysis

The noise module estimates how road traffic noise propagates around the selected site.

### Base Model

```text
L = L₀ − 20 log₁₀(r)
```

Where:

```text
L₀ = source emission level
r  = distance from the source
```

### Road Source Levels

| Road Class | Source Level |
|---|---:|
| Motorway | 78 dB |
| Primary | 72 dB |
| Secondary | 68 dB |
| Tertiary | 64 dB |
| Residential | 58 dB |

The model also applies corrections related to heavy vehicles, building barriers, ground absorption, and reflections. The final result is represented as a 50 m × 50 m grid-based noise heatmap.

---

# Automated PDF Report

The `/report` endpoint combines the individual analyses into one PDF document.

### Report Structure

```text
1. Cover / Site Information
2. Context & Zoning
3. Walking Accessibility
4. Driving Analysis
5. Public Transport
6. 360° View Classification
7. Noise Assessment
```

The documented report is generated as a seven-page PDF including the cover page.

---

# REST API

## Base URL

```text
https://automated-site-analysis-api.onrender.com
```

The project documentation identifies this as the deployed API endpoint.

## Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | API status |
| GET | `/search` | Location / lot search |
| GET | `/lot-boundary` | Site boundary retrieval |
| POST | `/walking` | Walking accessibility analysis |
| POST | `/driving` | Driving accessibility analysis |
| POST | `/transport` | Public transport analysis |
| POST | `/context` | Context and zoning analysis |
| POST | `/view` | 360° view classification |
| POST | `/noise` | Noise propagation analysis |
| POST | `/report` | Complete PDF report |

The documented POST analysis routes return PNG images, while `/report` returns a PDF stream.

---

# Example Request

```json
{
  "lot_id": "IL 1657"
}
```

### Walking Analysis

```bash
curl -X POST https://automated-site-analysis-api.onrender.com/walking \
  -H "Content-Type: application/json" \
  -d '{"lot_id":"IL 1657"}' \
  -o walking.png
```

### Complete Site Report

```bash
curl -X POST https://automated-site-analysis-api.onrender.com/report \
  -H "Content-Type: application/json" \
  -d '{"lot_id":"IL 1657"}' \
  -o site-report.pdf
```

The repository documentation provides the same API workflow for the individual analysis endpoints and the combined report endpoint.

---

# Python Client

```python
import requests

BASE_URL = "https://automated-site-analysis-api.onrender.com"


def fetch_analysis(endpoint: str, lot_id: str, output_path: str):
    response = requests.post(
        f"{BASE_URL}/{endpoint}",
        json={"lot_id": lot_id},
        timeout=30
    )

    response.raise_for_status()

    with open(output_path, "wb") as file:
        file.write(response.content)


# Generate walking analysis
fetch_analysis(
    "walking",
    "IL 1657",
    "walking.png"
)

# Generate complete report
fetch_analysis(
    "report",
    "IL 1657",
    "site-report.pdf"
)
```

---

# Local Setup

## Prerequisites

```text
Python 3.11+
pip
Git
```

## Installation

```bash
git clone https://github.com/your-username/GeoVision-AI-Powered-Spatial-Intelligence-Platform.git

cd GeoVision-AI-Powered-Spatial-Intelligence-Platform

pip install -r requirements.txt
```

## Start the Development Server

```bash
uvicorn app:app --host 0.0.0.0 --port 10000 --reload
```

The local API runs on:

```text
http://localhost:10000
```

Swagger documentation is available at:

```text
http://localhost:10000/docs
```

The uploaded project specifies Python 3.11+, pip, Git, and Uvicorn for local development.

---

# Deployment

The project is configured for deployment on Render through `render.yaml`.

### Deployment Configuration

```text
Runtime: Python
Build Command: pip install -r requirements.txt
Start Command: uvicorn app:app --host 0.0.0.0 --port 10000
```

The repository documentation also provides a Docker-based deployment option using Python 3.11.

---

# Performance Optimization

Several optimizations are included in the implementation.

### Dataset Reduction

The building dataset was reduced from more than 342,000 records to approximately 42,000 records by removing unnecessary attributes and preparing height information in advance.

### CRS Standardization

Datasets are prepared in EPSG:3857 to reduce repeated coordinate transformations during requests.

### Startup Preloading

GeoPackage datasets are loaded during API startup instead of being reloaded for every request.

### In-Memory Caching

Repeated requests for the same lot can reuse previously generated results.

### Output Optimization

Visualization output uses 200 DPI rather than the previous 400 DPI configuration to reduce processing and response size.

### Approximate Response Profile

| Request Type | Typical Time |
|---|---:|
| Cold Start | 10–15 sec |
| Warm Request | 5–10 sec |
| Cached Request | < 1 sec |

These timings are documented as the project's observed performance profile.

---

# Error Responses

| Status Code | Description |
|---|---|
| `200` | Analysis completed successfully |
| `422` | Invalid or unresolved lot identifier |
| `500` | Spatial processing or OSM extraction failure |
| `503` | Government GIS service unavailable |

The project documents these response conditions for the API.

---

# Project Status

## Implemented

```text
✓ Lot / Site Resolution
✓ OpenStreetMap Integration
✓ Pedestrian Network Analysis
✓ Driving Network Analysis
✓ Public Transport Analysis
✓ Context & Zoning Mapping
✓ 360° View Classification
✓ Noise Propagation Modelling
✓ Spatial Visualization
✓ PDF Report Generation
✓ Dataset Optimization
✓ In-Memory Caching
✓ Structured Logging
✓ Render Deployment
```

The source project documents these capabilities as completed implementation stages.

---

# Future Enhancements

Planned improvements include:

- API authentication
- Rate limiting
- Batch processing of multiple sites
- Background processing for long-running reports
- Persistent cloud storage
- React-based interactive dashboard
- Mapbox-based visualization
- Scalable cloud deployment

These are listed as future enhancements in the source project.

---

# What This Project Demonstrates

GeoVision brings together several areas of software and spatial engineering:

```text
Python Backend Development
          +
REST API Design
          +
GIS Data Processing
          +
Geospatial Computing
          +
Network Graph Analysis
          +
Spatial Visualization
          +
Environmental Modelling
          +
Automated Report Generation
          ↓
     GeoVision Platform
```

The modular design makes it possible to add additional spatial-analysis capabilities without redesigning the complete backend.

