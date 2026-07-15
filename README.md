# Benches for Gwinnett

Every Ride Gwinnett bus stop, scored by who has to permit a bench, whether there is already a sidewalk, and how long you wait.

**Live map:** https://matta174.github.io/benches-for-gwinnett/

Gwinnett runs 872 bus stops across 9 local routes. The median stop sees 28 bus trips per weekday, so a rider arriving at random waits about 15 minutes. Most of them wait standing. This repo is the evidence base for fixing that with wooden benches, following a model that worked in Chattanooga, Nashville, Berkeley, and metro Atlanta.

The headline finding: **Gwinnett County DOT controls the right of way at 557 of 872 stops, and 506 of those already have a sidewalk.** 58% of the system.

## What is here

```
index.html                      the map, single file, no build step
data/
  gwinnett_bus_stop_census.csv  all 872 stops, 18 columns
```

## Where the data came from

Everything is public. Nothing here is scraped, licensed, or private.

| What | Source | Link |
|---|---|---|
| Stop names, coordinates, routes, service levels | **Ride Gwinnett GTFS**. Feed version July 2 2026, valid through Dec 26 2026. 872 stops, 9 routes, 2,792 trips, 182,260 stop times. | [gtfs-zip.ashx](https://realtimegwinnett.availtec.com/InfoPoint/gtfs-zip.ashx) |
| Right of way authority, speed limits, road class, road names | **Gwinnett County GIS, Street Centerlines** (layer 2). 41,193 current segments. The `MAINTBY` field is a coded domain: 1 county, 2 municipal, 3 state, -1 private, -2 other. `STROUTE` and `FEDROUTE` identify state and federal routes. | [FeatureServer/2](https://services3.arcgis.com/RfpmnkSAQleRbndX/arcgis/rest/services/GwinnettTransportation/FeatureServer/2) |
| Sidewalk presence | **Gwinnett County GIS, Transportation Major Polygon** (layer 13). Planimetric polygons digitized from aerial photography. Feature code `TRANS_CODE_MAJ_PLY = 658` is Public Sidewalks. 197,007 polygons countywide. | [FeatureServer/13](https://services3.arcgis.com/RfpmnkSAQleRbndX/arcgis/rest/services/GwinnettTransportation/FeatureServer/13) |
| City limits | **Gwinnett County GIS, City_Area**. 19 municipal polygons. | [City_Area/FeatureServer/0](https://services3.arcgis.com/RfpmnkSAQleRbndX/arcgis/rest/services/City_Area/FeatureServer/0) |
| County boundary | **Gwinnett County GIS, countyboundary**. | [countyboundary/FeatureServer/0](https://services3.arcgis.com/RfpmnkSAQleRbndX/arcgis/rest/services/countyboundary/FeatureServer/0) |
| Cross reference for stop IDs | **Ride Gwinnett Transit Stops**, open data hub. 916 points, updated June 2026. Not used for the analysis, but useful for checking the GTFS against the county's own layer. | [open data hub](https://gcgis-gwinnettcountyga.hub.arcgis.com/datasets/612abc7590d9471191a686c21a6b1301_6/explore) |

Portals: [Gwinnett County Open Data](https://gcgis-gwinnettcountyga.hub.arcgis.com/) and [Ride Gwinnett](https://www.gwinnettcounty.com/government/departments/transportation/gwinnett-county-transit).

Basemap tiles are CARTO, map data is OpenStreetMap contributors.

Data pulled **July 15 2026**. GTFS changes about three times a year, so service numbers go stale. The GIS layers change continuously.

## Columns

| Column | Type | Meaning |
|---|---|---|
| `stop_id` | int | GTFS stop ID. Matches the ID on the pole. |
| `stop_name` | text | GTFS stop name, as Ride Gwinnett writes it. |
| `city` | text | City limits the stop falls inside, or `UNINCORPORATED`. Point in polygon against City_Area. |
| `jurisdiction` | text | Who maintains the road: `Gwinnett County DOT`, `GDOT (state route)`, `City`, `DeKalb County (outside Gwinnett)`, `Private`, `Unclear`. |
| `permit_path` | text | Plain language version of the same thing, matching the map legend. |
| `permit_difficulty` | text | `easiest`, `medium`, `hard`, `out of scope`, `unknown`. Sort on this. |
| `road` | text | Nearest street centerline `FULLNAME`. |
| `state_route` | float | State route number if the stop is on one. 13 is Buford Hwy, 8 is US 29 Lawrenceville Hwy, 140 is Jimmy Carter, 378 is Beaver Ruin, 10 is US 78. Blank means a local street. |
| `speed_limit` | float | Posted limit on that centerline. `0` means not recorded (14 stops, all in DeKalb). |
| `routes` | text | Comma separated route short names serving the stop. |
| `trips_per_weekday` | float | Bus trips stopping here on an average weekday. **This is service, not ridership.** |
| `has_sidewalk_15ft` | bool | A public sidewalk polygon exists within 15 ft of the stop point. |
| `sidewalk_polygons_within_50ft` | int | Count of sidewalk polygons within 50 ft. `0` means nothing nearby. |
| `priority` | float | A crude sort key, not a measurement. See below. |
| `lat`, `lon` | float | WGS84, straight from GTFS. |
| `streetview` | url | Opens Street View at the stop. |
| `gmaps` | url | Opens Google Maps at the stop. |

## How each field was derived

**Service levels.** GTFS `calendar_dates` maps each `service_id` to specific dates. Ride Gwinnett uses one service per day of week. Trips were counted against a representative Mon to Sun window (Sept 14 to 20, 2026), joined to `stop_times`, then weekday trips divided by 5. Every stop has Saturday service. None has Sunday.

**Jurisdiction.** Each stop was projected to EPSG:2240 (Georgia West, feet) and matched to the nearest street centerline within 150 ft via an STRtree. That segment's `MAINTBY` gives the maintainer. Any stop on a segment with a `STROUTE` or `FEDROUTE` value is classified GDOT regardless of `MAINTBY`, because the county's own layer codes most state routes as "Other (Non-County)" rather than "State". Stops with no centerline within range are outside Gwinnett, all of them DeKalb.

**Sidewalk.** One spatial query per stop against layer 13, filtered to feature code 658, at 15 ft and 50 ft radius. 872 queries.

**Priority.** `service_score x difficulty_tier x has_sidewalk`, where difficulty tier is county 3, city 2, GDOT 1, out of scope 0. It is a convenience for sorting the map, invented for that purpose, and it encodes an opinion (that easy permits matter as much as busy stops). Do not cite it. Sort on the real columns instead.

## Read this before you trust it

- **Sidewalk means a polygon exists nearby.** It says nothing about width, condition, connectivity, or whether there is room for a bench without blocking the pedestrian route. That last one is exactly what got Nashville's benches confiscated. 90% of stops passing this test is a desk check upper bound, not a green light.
- **GTFS coordinates drift.** A stop point can sit 30 to 60 ft off the actual pole, which matters when the test radius is 15 ft.
- **Sidewalk polygons come from aerial photography.** Tree canopy hides walks. Driveway aprons get over captured.
- **Trips per weekday is not ridership.** It tells you how long you wait, not how many people wait. Ride Gwinnett has automatic passenger counter data. Getting it would sharpen this a lot and is an open records request under O.C.G.A. 50-18-71.
- **Nothing here has been ground truthed.** Every stop needs eyes on it before anything gets built.

## Prior art

- [Chattanooga Urbanist Society](https://urbanistsociety.com/resources/), 60+ benches, and they publish their [bench guide](https://urbanistsociety.com/wp-content/uploads/2023/04/Chattanooga-Bench-Guide.pdf)
- To Nashville With Love and Benches For All, 50+ benches placed, about 17 collected by NDOT for obstructing the right of way, now working toward a permit pathway through NDOT's Tactical Urbanism Program
- [MARTA Army](https://www.martaarmy.org/), metro Atlanta, 501(c)(3). Their [Bus Stop Census](https://www.martaarmy.org/stop-census) is the methodology this borrows from, and they have crowdfunded transit amenities with Chamblee, Doraville, Clarkston, East Point, and Brookhaven.

## License

MIT for the code.

The census is derived from public GTFS and county GIS. It is mostly uncopyrightable fact. Take it, fork it, do this in your county. That is the point.
