# DuckDB Spatial Extension

## Introduction

The DuckDB Spatial Extension adds geospatial processing capabilities directly into DuckDB, letting you work with locations, maps, and shapes right alongside your regular data. It provides a `GEOMETRY` type, a rich set of spatial functions, coordinate transformations, and support for reading and writing many GIS formats through GDAL. This guide walks through the core concepts—geometry types, coordinate systems, spatial functions, and spatial joins—and ties them together with a hands-on NYC taxi example.

---

## Chapter 1: What is the DuckDB Spatial Extension?

DuckDB is an analytical database.

Normally, DuckDB understands:

- Numbers
- Text
- Dates
- JSON

The Spatial Extension teaches DuckDB about:

- Locations
- Maps
- Routes
- Buildings
- Polygons
- GPS Coordinates

Install it with:

```sql
INSTALL spatial;
LOAD spatial;
```

After loading, DuckDB can process spatial data just like PostGIS.

---

## Chapter 2: What is Geospatial Data?

Geospatial data is data that contains a location on Earth.

Examples:

| Location | Latitude | Longitude |
| --- | --- | --- |
| Charlotte | 35.2271 | -80.8431 |
| New York | 40.7128 | -74.0060 |
| Building Entrance | 35.2000 | -80.8000 |

Real-world examples:

- POIs
- Buildings
- Roads
- Flight Routes
- Parking Lots
- Sensors
- Cities

---

## Chapter 3: The Three Core Geometry Types

Everything in GIS starts with:

### Point

A single location.

Example:

```sql
SELECT ST_Point(-80.8431, 35.2271);
```

Visual:

```
•
```

Examples:

- POI
- Sensor
- Building Entrance
- Parking Space

### Line

A path connecting points.

Example:

```sql
SELECT ST_MakeLine(
    ST_Point(0,0),
    ST_Point(10,10)
);
```

Visual:

```
•------------•
```

Examples:

- Walking Route
- Taxi Route
- Flight Path
- Road

### Polygon

An enclosed area.

Visual:

```
+-----------+
| Building  |
+-----------+
```

Examples:

- Building Boundary
- Campus Boundary
- Parking Lot
- City Boundary

---

## Chapter 4: Understanding Coordinate Systems

This is where most beginners get confused.

### EPSG:4326

Standard GPS coordinates.

Example:

```
Latitude
Longitude
```

Like:

```
35.2271
-80.8431
```

GPS, Google Maps, and mobile devices commonly use this system.

### ESRI:102718

Used in the article's NYC example.

Instead of:

```
Latitude/Longitude
```

it uses:

```
Projected Coordinates
```

which are better for accurate distance measurements. The article transforms from WGS84 (EPSG:4326) to ESRI:102718 before calculating distances and performing joins with the taxi zones.

### Why Transform?

Imagine:

```
Point = GPS Coordinates
Zone Polygon = ESRI Coordinates
```

They cannot be compared directly.

So DuckDB converts:

```sql
ST_Transform(
    point,
    'EPSG:4326',
    'ESRI:102718'
)
```

Meaning:

```
Convert GPS
      ↓
To ESRI coordinates
```

---

## Chapter 5: Useful Spatial Functions

### ST_Point()

Creates a point.

```sql
ST_Point(lat, lon)
```

Example:

```sql
ST_Point(35.2, -80.8)
```

### ST_MakeLine()

Creates a route between two points.

```sql
ST_MakeLine(start, end)
```

Visual:

```
A ----------- B
```

### ST_Distance()

Calculates distance.

```sql
ST_Distance(point1, point2)
```

Examples:

- Building to Building
- POI to POI
- Device to Device

### ST_Within()

Checks if a point is inside a polygon.

Example:

```sql
ST_Within(point, polygon)
```

Visual:

```
+---------+
|    •    |
+---------+
```

Result:

```
TRUE
```

### ST_Area()

Calculates area.

```sql
ST_Area(polygon)
```

Examples:

- Building Size
- Floor Size
- Parking Lot Size

---

## Chapter 6: Reading GIS Files

One of the strongest features of the Spatial Extension is support for many formats through GDAL.

Common formats:

- GeoJSON
- GeoPackage
- Shapefile
- KML
- WKB
- WKT

Read a file:

```sql
SELECT *
FROM ST_Read('buildings.geojson');
```

Think of `ST_Read()` as:

```sql
read_csv()
```

but for maps.

---

## Chapter 7: Taxi Example Walkthrough

The article demonstrates the Spatial Extension using NYC taxi ride data.

### Example Data

The sample data used in this walkthrough comes from the official DuckDB Spatial repository:

👉 [duckdb-spatial/test/data/nyc_taxi](https://github.com/duckdb/duckdb-spatial/tree/main/test/data/nyc_taxi)

It contains:

- **`yellow_tripdata_2010-01-limit1mil.parquet`** — ~1 million yellow-cab trip records with pickup/dropoff coordinates, distance, and timestamps.
- **`taxi_zones/`** — the NYC taxi-zone polygons (a Shapefile) used for the spatial joins.

You can load both directly in DuckDB:

```sql
INSTALL spatial;
LOAD spatial;

-- Trips (Parquet)
SELECT *
FROM read_parquet('yellow_tripdata_2010-01-limit1mil.parquet')
LIMIT 5;

-- Taxi-zone polygons (Shapefile via GDAL)
SELECT *
FROM ST_Read('taxi_zones/taxi_zones.shp')
LIMIT 5;
```

### Step 1: Load Taxi Data

Input:

- Pickup Coordinates
- Dropoff Coordinates
- Distance
- Time

### Step 2: Create Geometry

Convert coordinates into points:

```sql
ST_Point(...)
```

Result:

```
pickup_point
dropoff_point
```

### Step 3: Calculate Aerial Distance

```sql
ST_Distance(...)
```

Visual:

```
Bird Route
A -------- B
```

This is the shortest possible route.

### Step 4: Compare Actual Distance

Example:

```
Taxi Distance = 5 miles
Aerial Distance = 3 miles
```

Difference:

```
2 miles
```

Normal.

Bad record:

```
Taxi Distance = 2 miles
Aerial Distance = 5 miles
```

Impossible.

Remove it.

---

## Chapter 8: Spatial Joins

Spatial joins are one of the most important GIS concepts.

Normal SQL Join:

```sql
customer.id = orders.customer_id
```

Spatial Join:

```
Point inside Polygon
```

using:

```sql
ST_Within(point, polygon)
```

The article uses this to find:

- Starting Taxi Zone
- Ending Taxi Zone

For example:

```
Pickup Point -> Manhattan

Dropoff Point -> Queens
```

The article joins rides to taxi-zone polygons using `ST_Within()` after transforming coordinates into the same projection.

---

## Chapter 9: Creating Routes

Once pickup and dropoff points are available:

```sql
ST_MakeLine(
    pickup_point,
    dropoff_point
)
```

creates:

```
Taxi Route
```

Visual:

```
Pickup * ----------- * Dropoff
```

---

## Chapter 10: Why ST_FlipCoordinates()?

The article uses:

```sql
.ST_FlipCoordinates()
```

because coordinates were created as:

```
Latitude, Longitude
```

Many GIS systems expect:

```
Longitude, Latitude
```

Before:

```
35.2, -80.8
```

After:

```
-80.8, 35.2
```

---

## Chapter 11: What is WKB?

WKB means:

```
Well-Known Binary
```

A compact binary representation of geometry. The article converts route geometries to WKB before exporting.

Example:

```sql
ST_AsWKB(...)
```

Think:

```
Geometry
     ↓
Binary Format
```

---

## Chapter 12: GeoJSON vs GeoJSONSeq

### GeoJSON

```json
[
  {...},
  {...}
]
```

All features stored in one JSON array.

### GeoJSONSeq

```
{...}
{...}
{...}
```

One feature per line.

Benefits:

- Better for large datasets
- Faster streaming
- Easier processing

The article exports to GeoJSONSeq using GDAL.

---

## Chapter 13: Exporting Results

The final step:

```sql
COPY (...)
TO 'joined.geojsonseq'
WITH (
    FORMAT GDAL,
    DRIVER 'GeoJSONSeq'
);
```

Meaning:

```
Take all routes
      ↓
Save as GeoJSONSeq
      ↓
Open in GIS tools
```

---

## Chapter 14: Viewing GeoJSONSeq in DuckDB

Read it back:

```sql
SELECT *
FROM ST_Read('joined.geojsonseq');
```

Show geometry:

```sql
SELECT
    ST_AsText(geom)
FROM ST_Read('joined.geojsonseq');
```

Check geometry type:

```sql
SELECT
    ST_GeometryType(geom)
FROM ST_Read('joined.geojsonseq');
```

Expected:

```
LINESTRING
```
