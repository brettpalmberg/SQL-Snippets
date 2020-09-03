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
