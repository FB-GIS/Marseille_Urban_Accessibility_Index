# Urban Accessibility Index — Marseille

A geospatial analysis pipeline that computes a simple composite **Urban Accessibility Index** for the city of Marseille, France. The index aggregates four urban mobility and amenity indicators — schools, subway stations, bike paths, and parks — and maps them at two geographic resolutions: **administrative districts** (arrondissements) and **H3 hexagonal grid cells**.

## Overview

Urban accessibility — how easily residents can reach essential services.  
This project provides a reproducible, open-data pipeline to score and compare accessibility across Marseille's 16 arrondissements and their constituent H3 hexagonal cells.  

All input data is fetched live from **OpenStreetMap** via the `osmnx` library. No proprietary data sources are required.  

Districts Visualization: 
![alt text](https://github.com/FB-GIS/Marseille_Urban_Accessibility_Index/blob/main/public/districts.png)

H3 Visualization: 
![alt text](https://github.com/FB-GIS/Marseille_Urban_Accessibility_Index/blob/main/public/h3.png)

---

## Methodology  

The pipeline proceeds in two parallel stages:  

### Stage 1 — District-Level Analysis  

1. Fetch the administrative boundaries of Marseille's 16 arrondissements.  
2. For each district, count or measure the four indicators using **geometric intersection**.  
3. Apply **Min-Max normalization** to bring all indicators to a [0, 1] scale.  
4. Apply **spatial smoothing** by averaging each district's values with those of its directly adjacent neighbors (touching geometries).  
5. Re-normalize and compute a **composite score** (sum of four indicators, range 0–4).  
6. Visualize the result as a choropleth map using `leafmap`.  

### Stage 2 — Hexagonal Grid Analysis (H3 Resolution 9)  

1. Polyfill each arrondissement with **H3 hexagonal cells** at resolution 9 (~105 m edge length, ~0.1 km²).  
2. For each hexagon, measure the four indicators using **circular buffers**:  
   - **1,600 m radius** for schools, subway stations, and bike paths (~20-min walk)  
   - **800 m radius** for parks (~10-min walk)  
3. Apply Min-Max normalization.  
4. Apply **H3 neighbor smoothing** using `h3.grid_disk(cell, k=2)` — each cell is averaged over its 2-ring neighborhood (up to 19 cells).  
5. Re-normalize and compute the composite score.  
6. Export results to **GeoJSON** and visualize interactively.  

### Indicators  

| Indicator | Measurement | Buffer (hex stage) |
|---|---|---|  
| Schools | Count | 1,600 m |  
| Subway stations | Count | 1,600 m |  
| Bike paths | Total length (m) | 1,600 m |  
| Parks | Total area (m²) | 800 m |  

---

### Score interpretation

| Score range | Accessibility level |  
|---|---|  
| 3.5 – 4.0 | Very high |  
| 2.5 – 3.5 | High |  
| 1.5 – 2.5 | Moderate |  
| 0.5 – 1.5 | Low |  
| 0.0 – 0.5 | Very low |  

---

## Dependencies  

```
geopandas>=0.14  
h3>=4.0.0  
h3pandas>=0.2  
leafmap>=0.30  
osmnx>=1.9  
scikit-learn>=1.3  
numpy>=1.24  
pandas>=2.0  
shapely>=2.0  
```

## Limitations

- **Data completeness:** Results depend on OpenStreetMap coverage, which varies across districts. Less-mapped areas may show artificially low scores.  
- **Equal weighting:** All four indicators contribute equally to the composite score. No evidence-based weighting is applied.  
- **Static snapshot:** The analysis reflects OSM data at the time of execution. Results may vary if re-run at a later date.  
- **Buffer simplification:** The 1,600 m and 800 m buffers are Euclidean (straight-line) distances, not network-based travel distances. They do not account for physical barriers such as highways or rail lines.  
- **Hex resolution:** H3 resolution 9 provides fine-grained results but significantly increases computation time. Lower resolutions (7 or 8) can be used for faster prototyping.  
 
