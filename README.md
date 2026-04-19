# Farm for the Future — Green Carbon Track B

**Automated farmer-to-land matching system for J-Credit carbon registry compliance**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-farmmatchertool.netlify.app-brightgreen)](https://farmmatchertool.netlify.app/)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-deployed-blue)](https://saadt127.github.io/Minerva-Hackathon-Track-B/)

---

## Overview

This project solves a critical real-world logistics problem in sustainable agriculture: assigning **103 smallholder farmers** to **42 hand-drawn polygon fields** across five field groups (A–E), where the two data sources — a farmer list and a KMZ polygon map — share no common identifiers.

Built for the **Minerva Hackathon Green Carbon Track B**, the system pairs rigorous algorithmic matching with a human-in-the-loop confidence scoring layer, ensuring every assignment is both defensible and traceable for J-Credit carbon certification.

---

## Live Demo

**[https://saadt127.github.io/Minerva-Hackathon-Track-B/](https://saadt127.github.io/Minerva-Hackathon-Track-B/)**

The interactive dashboard lets you explore:
- Farmer-to-polygon assignments on a live Leaflet map
- Confidence scores for each assignment
- A resolution queue flagging low-confidence cases for field-staff review
- Cluster visualizations showing how polygons were spatially grouped

---

## Problem Statement

| Dimension | Detail |
|-----------|--------|
| Farmers | 103 smallholders across 5 field groups (A–E) |
| Polygons | 42 hand-drawn fields in a KMZ map (no group labels) |
| Average sharing ratio | ~2.5 farmers per polygon |
| Challenge | Two datasets with no shared keys — matching must be inferred purely from area measurements and spatial geometry |

---

## Algorithm

The matching is solved in two hierarchical stages:

### Stage 1 — Spatial Clustering
Unlabeled polygons are grouped into 5 clusters using **weighted K-means** on polygon centroids (area-weighted), then matched to the 5 farmer group area targets via the **Hungarian algorithm** (minimum total area mismatch).

### Stage 2 — Within-Group Assignment
Inside each group, farmers are assigned to polygons using a **min-cost flow network** that:
- Handles many-to-one relationships (multiple farmers sharing one polygon)
- Iteratively refines per-farmer polygon share estimates over 3 passes
- Minimises a normalised area-difference cost: `|Δarea| / max(farmer_area, polygon_share)`

### Confidence Scoring
Each assignment receives a score in [0, 1]:

```
confidence = tanh(5 × (second_best_cost − best_cost))
```

Assignments below a 0.25 threshold are automatically routed to a human review queue.

### Stability Validation
Bootstrap resampling validates matching robustness under 2–20% measurement noise (multiplicative Gaussian perturbation), producing stability curves without requiring ground-truth labels.

---

## Repository Structure

```
Minerva-Hackathon-Track-B/
├── index.html              # Interactive dashboard (Leaflet map, charts, review queue)
├── notebook/
│   └── farm_matcher_real.ipynb   # Full Python matching algorithm
├── data/
│   ├── farmer_list.xlsx    # 103 farmers with areas and group assignments
│   ├── polygon_areas.xlsx  # 42 polygons with calculated areas
│   └── project_area.kmz   # Geospatial polygon data
└── README.md
```

---

## Tech Stack

### Algorithm (Python)
| Library | Role |
|---------|------|
| `networkx` | Min-cost flow graph construction and solving |
| `scikit-learn` | Weighted K-means clustering |
| `scipy.optimize` | Hungarian algorithm (cluster-to-group assignment) |
| `geopandas` / `shapely` | Geospatial geometry and UTM reprojection (EPSG:32648) |
| `folium` | Interactive assignment map generation |
| `pandas` / `numpy` | Data processing and matrix operations |

### Dashboard (Frontend)
| Technology | Role |
|------------|------|
| Leaflet.js | Interactive polygon map |
| CSS custom properties | Animated, responsive UI |
| Embedded SVG filters | Visual styling |

---

## Key Design Decisions

**Why two-stage matching?**
Polygons carry no group labels. Clustering spatially first mirrors how a field agent would manually group plots before making fine-grained assignments — it is more interpretable and auditable than a flat 103×42 optimisation.

**Why min-cost flow over Hungarian?**
The Hungarian algorithm enforces strict 1:1 matching. Flow networks natively model the shared-plot reality where one polygon may be cultivated by multiple farmers.

**Why iterative share refinement?**
The number of farmers sharing each polygon is not known a priori. Three iterations of alternating assignment → share recalculation converges quickly and avoids hard-coded assumptions.

**Why confidence scoring?**
J-Credit traceability standards require transparent uncertainty signals. Routing low-confidence assignments to human review rather than forcing automated decisions reduces audit risk.

---

## Running the Algorithm Locally

```bash
# Install dependencies
pip install pandas numpy scipy scikit-learn geopandas shapely networkx folium openpyxl

# Open the notebook
jupyter notebook notebook/farm_matcher_real.ipynb
```

Input files (`data/`) are loaded automatically. Outputs written to the working directory:
- `assignments.csv` — all 103 farmer–polygon links with confidence scores
- `flagged_for_review.csv` — low-confidence assignments for field staff
- `assignment_map.html` — interactive Folium map

---

## Authors

Built for the **Minerva Hackathon — Green Carbon Track B**
