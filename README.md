# SQL-Snippets

Many of these originally come from questions answered by others on StackOverflow or similar.

## Working With PostGIS Geometries

Select List of Features from GeoJSON FeatureCollection
```
WITH DATA AS
  (SELECT '{"type":"FeatureCollection","features":[...'::JSON AS fc)
SELECT row_number() OVER () AS gid,
       ST_AsText(ST_GeomFromGeoJSON(feat->>'geometry')) AS geom,
       feat->'properties' AS properties
FROM
  (SELECT json_array_elements(fc->'features') AS feat
   FROM DATA) AS f;
```

Update Geometry Field with Collection Created from FeatureCollection
```
WITH FC as (SELECT ST_AsText(ST_Collect(ST_GeomFromGeoJSON(feat->>'geometry'))) AS featuretext, 'c785f4de-ab17-444b-b6e6-6f1ad16676e8'::uuid AS id
FROM (
  SELECT json_array_elements('{-- GEOJSON HERE --}'::json->'features') AS feat) AS f
)
UPDATE table1
SET geometry = FC.featuretext
FROM FC
WHERE fc.id = table1.id
```

Getting shapefiles into PostGIS tables. Original shapefiles converted to PostGIS Insert Statements using `shp2pgsql`
```
INSERT INTO subbasin
SELECT uuid_generate_v4() AS id,
       a.basin_id::uuid AS basin_id,
       a.name AS name,
       a.geometry
FROM (
	SELECT gg.name AS name,
	       gg.basin_id AS basin_id, 
	       ST_Transform(
				gg.geom,
				'+proj=aea +lat_0=23 +lon_0=-96 +lat_1=29.5 +lat_2=45.5 +x_0=0 +y_0=0 +datum=NAD83 +units=us-ft +no_defs',
				5070
		   ) AS geometry
	FROM (
			    SELECT '{c54eab5b-1020-476b-a5f8-56d77802d9bf}' AS basin_id, geom, hmsnm AS name FROM lower_cumbtenn_basins_ver1_shg_ft
		UNION SELECT '{c785f4de-ab17-444b-b6e6-6f1ad16676e8}' AS basin_id, geom, hms_name AS name FROM upper_cumb_basins_ver3_shg_ft
	) gg
) a
```
