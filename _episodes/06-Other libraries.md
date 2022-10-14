---
title: "Other method of plotting with Geospatial data"
teaching: 20
exercises: 0
questions:
- "How to train a Machine Learning model with Categorical output?"
objectives:
- "Learn different ML Supervised Learning with Categorical output"
keypoints:
- "Decision Tree, Random Forest"
---

# 6 Other libraries for Geospatial data analytics

## 6.1 Folium

Folium builds on the data wrangling strengths of the Python ecosystem and the mapping strengths of the leaflet.js library. Manipulate your data in Python, then visualize it in on a Leaflet map via folium.

folium makes it easy to visualize data that’s been manipulated in Python on an interactive leaflet map. It enables both the binding of data to a map for choropleth visualizations as well as passing rich vector/raster/HTML visualizations as markers on the map.

The library has a number of built-in tilesets from OpenStreetMap, Mapbox, and Stamen, and supports custom tilesets with Mapbox or Cloudmade API keys. folium supports both Image, Video, GeoJSON and TopoJSON overlays.

Contents

### Install

```python
pip install folium
```

### Basemap

Get your current location

```python
import geocoder
myloc = geocoder.ip('me')
my_lat,my_lon=myloc.latlng
```

#### Plotting regular map

```python
#Regular map
map = folium.Map(location=[my_lat,my_lon])
map
```

![image](https://user-images.githubusercontent.com/43855029/179088395-7367b3f2-9313-4ddf-b50f-72ef7d42c4aa.png)

Zooming in

```python
#Regular map
map = folium.Map(location=[my_lat,my_lon], zoom_start=15)
map
```
![image](https://user-images.githubusercontent.com/43855029/179088472-0f33b6e9-4a5a-42c8-86aa-07e326a159ae.png)

#### Stamen Terrain
```python
map = folium.Map(location = [my_lat,my_lon], tiles = "Stamen Terrain", zoom_start = 9)
map
```
![image](https://user-images.githubusercontent.com/43855029/179088587-6b41b6fe-f652-453c-963f-016f06c21a6b.png)

####  OpenStreetMap

```python
map = folium.Map(location = [my_lat,my_lon], tiles='OpenStreetMap' , zoom_start = 10)
map
```

![image](https://user-images.githubusercontent.com/43855029/179088662-60435f81-5360-43b0-982b-6212d96bf197.png)

#### Stamen Toner
```python
map = folium.Map(location = [my_lat,my_lon], tiles='Stamen Toner', zoom_start = 10)
map
```

![image](https://user-images.githubusercontent.com/43855029/179088720-b9b6b4e2-0be5-4e90-8556-98b4407ff0a1.png)

#### Plotting with Markers

```python
m = folium.Map(location=[45.372, -121.6972], zoom_start=12, tiles="Stamen Terrain")

tooltip = "Click me!"

folium.Marker(
    [45.3288, -121.6625], popup="<i>Mt. Hood Meadows</i>", tooltip=tooltip
).add_to(m)
folium.Marker(
    [45.3311, -121.7113], popup="<b>Timberline Lodge</b>", tooltip=tooltip
).add_to(m)

m
```
![image](https://user-images.githubusercontent.com/43855029/179088869-c5c5b578-f490-44ca-b9a6-3f6d7e4acf57.png)

More [information on using folium](https://python-visualization.github.io/folium/quickstart.html)
