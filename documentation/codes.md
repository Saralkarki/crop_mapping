---
title: Python codes
layout: default
permalink: '/codes'
---
<a href="/crop_mapping/Crop_Mapping_data_collection_Monitoring.ipynb" download> Download the Monitoring python code  </a>


## Getting data from the form via API
- How to get the data and the form id required for the survey from kobo

```python
data_url = "https://kc.humanitarianresponse.info/api/v1/data/yourform.json"
```

## Importing the required libraries to download the data, transform the data and map it

```python
import requests
import pandas as pd
import os
import geopandas as gpd
import shapely import wkt
import folium
import matplotlib.pyplot as plt
# if geopandas not installed
# pip install geopandas 
```
Note: Install the required python libraries

## Defining the username and password for the data server

```python
username = "USERNAMEGOESHERE"
password = 'PASSWORDGOESHERE'
```
## Reading the data from the server via API request

```python
response = requests.get(data_url, auth=(username, password))
j = response.json()
df = pd.json_normalize(j)
```

## Checking to see if data downloads

```python
df.shape # for seeing the shape of the dataset
```

## Checking to see all the column names (only select the columns that are required)

```python
df.columns

# defining the columns required
cols_required = ['enumerators','state','district','crop','PlotGPS','shape','shape_area','rounded_shape_area']
## filtering the data with only the data columns required for the analyis
df = df[cols_required]

```

## Quickly verifying how many dta has been collected by enumerators and taking a look at the crop

```python
# see the count by enumerators and crop
df.groupby(['enuemrators'])['crop'].counts()
# take a look at the dataset by crops
df['crop'].value_counts()
## Number of samples by enumerators and crop
df.groupby(['enumerators','crop'])['crop'].count()
```

![enuemrators](enumerators.png)

![enuemrator_crop](enumerators_crop.png)



## Point Visualization for monitoring

```python
def wkt_point(pon):
    point = "POINT ({} {})".format(pon[1], pon[0])
    return point
df['point_geom'] = df['geolocation'].map(wkt_point)
# df

df['point_geometry'] = df.point_geom.apply(wkt.loads)
df.drop('point_geom', axis=1, inplace=True) #Drop WKT column


gdf_point = gpd.GeoDataFrame(df, geometry='point_geometry')

```

##  Monitoring the feild polygons for each crop

```python
# https://python.hotexamples.com/examples/shapely.wkt/-/dumps/python-dumps-function-examples.html#0x346e25d2de439ed401c723d3f6e3e4c911cb28698b845c5bec33796af1b082ac-106,,134,
# https://github.com/Cadasta/cadasta-platform/blob/master/cadasta/xforms/utils.py

def odk_geom_to_wkt(coords):
    """Convert geometries in ODK format to WKT."""

    if coords == '':
        return ''
#     print(coords)
    if str(coords)!='nan':
        coords = coords.replace('\n', '')
        coords = coords.split(';')
        coords = [c.strip() for c in coords]
        if (coords[-1] == ''):
            coords.pop()

        if len(coords) > 1:
            # check for a geoshape taking into account
            # the bug in odk where the second coordinate in a geoshape
            # is the same as the last (first and last should be equal)
            if len(coords) > 3:
                if coords[1] == coords[-1]:  # geom is closed
                    coords.pop()
                    coords.append(coords[0])
            points = []
            for coord in coords:
                coord = coord.split(' ')
                coord = [x for x in coord if x]
                latlng = [float(coord[1]),
                          float(coord[0])]
                points.append(tuple(latlng))
            if (coords[0] != coords[-1] or len(coords) == 2):
                return dumps(LineString(points))
            else:
                return dumps(Polygon(points))
        else:
            latlng = coords[0].split(' ')
            latlng = [x for x in latlng if x]
            return dumps(Point(float(latlng[1]), float(latlng[0])))
    else:
        return np.nan
def wkt_loads(x):
    try:
        return wkt.loads(x)
    except Exception:
        return None

df['geom_poly'] = df['shape'].map(odk_geom_to_wkt)
df_new = df
df_new['geometry'] = df_new.geom_poly.apply(wkt_loads)
df_new.drop('geom_poly', axis=1, inplace=True) #Drop WKT column
df_new = df_new.dropna(subset=['geometry'])

# Geopandas GeoDataFrame
gdf_poly = gpd.GeoDataFrame(df_new, geometry='geometry')
```

## Map Visualization of the survey

```python
map_vis = folium.Map(location=[28.5212389, 81.0903013], zoom_start=10, tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
        attr="Google",
        name="Google Satellite")

# Create a geometry list from the GeoDataFrame
geo_df_list = [[point.xy[1][0], point.xy[0][0]] for point in gdf_point.point_geometry]

# Iterate through list and add a marker for each volcano, color-coded by its type.
i = 0
for coordinates in geo_df_list:

    # Place the markers with the popup labels and data
    map_vis.add_child(
        folium.Marker(
            location=coordinates,
            popup=
                "Crop: " + str(gdf_point.crop[i]) + "<br>"
                + "Enumerator: " + str(gdf_point.enumerators[i]) + "<br>"
            ),
        )
    i = i + 1

for _, r in gdf_poly.iterrows():
    # Without simplifying the representation of each borough,
    # the map might not be displayed
    sim_geo = gpd.GeoSeries(r['geometry']).simplify(tolerance=0.001)
    geo_j = sim_geo.to_json()
    geo_j = folium.GeoJson(data=geo_j,
                           style_function=lambda x: {'fillColor': 'orange'})
    folium.Popup(r['enumerators']).add_to(geo_j)
    geo_j.add_to(map_vis)

map_vis

```

![map visualization](/uploads/image_screenshots/map_vis.png)



