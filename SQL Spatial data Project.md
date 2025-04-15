# 🗺️ SQL Server Spatial Data Project

This project showcases my ability to use **SQL Server spatial features** with GEOGRAPHY data types, stored procedures, views, and triggers to manage and analyze geographic shapes such as points, polygons, and linestrings.

## 🧠 Project Summary

The project performs the following tasks:

1. **Create Tables** – Points, Polygons, Linestrings using `GEOGRAPHY`.
2. **Insert WKT Data** – WKT (Well-Known Text) shapes are inserted dynamically via stored procedures.
3. **Create Unified View** – A view to unify all shapes into a single result.
4. **Measure Route Length Difference** – Calculates and compares lengths of two routes.
5. **Buffer Analysis** – Identifies points inside or outside a specific polygon buffer.
6. **Trigger Protection** – Logs attempts to drop views and blocks them.

## 📸 Screenshots of Outputs

### ✅ View of All Shapes
A view combining all shape data from three tables.

![Spatial Data Output](./screenshots/spatial_data1.png)

### 📏 Route Distance Calculation
This procedure calculates and prints the distance difference between two linestrings.

![Route Difference](C:\Users\Abaan\OneDrive\Desktop\distance.png)

---

### 📍 Points in the spatial data
Shows points inside a defined polygon buffer using `STIntersects`.

![Points Inside](C:\Users\Abaan\OneDrive\Desktop\spatial result.png)


## 💬 How It Works

- All data is inserted using **stored procedures**.
- Geometry is parsed from **WKT format**.
- Logic uses `GEOGRAPHY::STGeomFromText`, `STLength`, and `STIntersects`.
- A **view** (`VW_ALL_SHAPES`) is created to unify data.
- A **trigger** (`TRG_PREVENT_VIEW_DROP`) logs and prevents view deletion.

---

## 🧪 Example Use

```sql
-- Create all spatial tables
EXEC UDP_CREATE_TABLES;

-- Insert sample shapes
EXEC UDP_INSERT_WKT 'POINT', 'POINT(175.6237343 -40.3689645)', 'Sample Point';

-- View all shapes
SELECT * FROM VW_ALL_SHAPES;

-- Check route length difference
EXEC UDP_ROUTE_DIFFERENCES_IN_METERS;

-- Check if points are within a buffer
EXEC UDP_DISJOINT 0; -- Inside
EXEC UDP_DISJOINT 1; -- Outside
