---
title: "Introduction to Raster data in Python"
teaching: 20
exercises: 0
questions:
- "What is Raster data format"
objectives:
- "Open raster data using Python."
- "Be able to list and identify 3 spatial attributes of a raster dataset: extent, crs and resolution."
- "Explore and plot the distribution of values within a raster using histograms."
- "Access metadata stored in a GeoTIFF raster file via TIF tags in Python."

keypoints:
- "raster"
---

In previous chapters you learned how to use the open source Python package **Geopandas** to open vector data stored in **shapefile** format. In this chapter you will learn how to use the open source Python packages **rasterio** combined with **numpy** and **earthpy** to open, manipulate and plot **raster** data in Python. In this chapter, you will learn how to open and plot a **lidar** raster dataset in Python. You will also learn about key attributes of a raster dataset:

- Spatial resolution
- Spatial extent and
- Coordinate reference systems

## 3.1. Introduction to Raster

### What is Raster?

- Raster or “gridded” data are stored as a grid of values which are rendered on a map as pixels. Each pixel value represents an area on the Earth’s surface. A raster file is composed of regular grid of cells, all of which are the same size.

- You’ve looked at and used rasters before if you’ve looked at photographs or imagery in a tool like Google Earth. However, the raster files that you will work with are different from photographs in that they are spatially referenced. Each pixel represents an area of land on the ground. That area is defined by the spatial resolution of the raster.

![image](https://user-images.githubusercontent.com/43855029/177869543-341464f3-bc32-4954-8835-c3a668d9c5e1.png)

### Raster Facts
A few notes about rasters:

- Each cell is called a pixel.
- And each pixel represents an area on the ground.
- The resolution of the raster represents the area that each pixel represents on the ground. So, a 1 meter resolution raster, means that each pixel represents a 1 m by 1 m area on the ground.
- A raster dataset can have attributes associated with it as well. For instance in a Lidar derived digital elevation model (DEM), each cell represents an elevation value for that location on the earth. In a LIDAR derived intensity image, each cell represents a Lidar intensity value or the amount of light energy returned to and recorded by the sensor.

![image](https://user-images.githubusercontent.com/43855029/177869683-ba6d31d7-c893-4a89-b251-2abd025a4349.png)

## 3.2 Working with raster in Python

Raster data can be used to store many different types of scientific data including

- elevation data
- canopy height models
- surface temperature
- climate model data outputs
- landuse / landcover data
and more...

In this lesson you will learn more about working with lidar derived raster data that represents both terrain / elevation data (elevation of the earth’s surface), and surface elevation (elevation at the tops of trees, buildings etc above the earth’s surface).

![image](https://user-images.githubusercontent.com/43855029/177870089-aebc3c6d-3791-4f37-98a9-198811e8a642.png)

To begin load the packages that you need to process your raster data.

Import necessary packages
```python
import os

import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
# Use geopandas for vector data and xarray for raster data
import geopandas as gpd
import rioxarray as rxr

import earthpy as et

# Prettier plotting with seaborn
sns.set(font_scale=1.5, style="white")
```

Download sample data and set working directory

```python
et.data.get_data("colorado-flood")
os.chdir(os.path.join(et.io.HOME,'earth-analytics'))
```

![image](https://user-images.githubusercontent.com/43855029/177871705-c7b9cb10-3c2d-409d-b5c0-ac41865a5c2c.png)

```python
dem_pre_path = os.path.join("data","colorado-flood",
                            "spatial",
                            "boulder-leehill-rd",
                            "pre-flood",
                            "lidar",
                            "pre_DTM.tif")

dtm_pre_arr = rxr.open_rasterio(dem_pre_path)
dtm_pre_arr
```

When you open raster data using **xarray** or **rioxarray** you are creating an **_xarray.DataArray_**. The _DataArray_ object stores the:

- raster data in a numpy array format
- spatial metadata including the CRS, spatial extent of the object
- and any metadata

**Xarray** and **numpy** provide an efficient way to work with and process raster data. **xarray** also supports **dask** and **parallel processing** which allows you to more efficiently process larger datasets using the processing power that you have on your computer

When you add **rioxarray** to your package imports, you further get access to spatial data processing using **xarray** objects. Below, you can view the spatial extent (bounds()) and CRS of the data that you just opened above.


```python
# View the Coordinate Reference System (CRS) & spatial extent
print("The CRS for this data is:", dtm_pre_arr.rio.crs)
print("The spatial extent is:", dtm_pre_arr.rio.bounds())
```

The nodata value (or fill value) is also stored in the xarray object.

```python
print("The no data value is:", dtm_pre_arr.rio.nodata)
```

Once you have imported your data, you can plot is using xarray.plot().

```python
dtm_pre_arr.plot()
plt.show()
```
![image](https://user-images.githubusercontent.com/43855029/177872105-545e2e5d-2f4f-4e47-a3f7-a6afe31a235e.png)

```
When a plot looks off, it is always a good idea to explore whether nodatavalues exist in your data. Often no data values are very large of negative numbers that are not likely to be valid values in your data. These values will skew any plots (or calculations) in your analysis.
```

The data above should represent terrain model data. However, the range of values is not what is expected. These data are for Boulder, Colorado where the elevation may range from 1000-3000m. There may be some outlier values in the data that may need to be addressed. Below you look at the distribution of pixel values in the data by plotting a histogram.

Notice that there seem to be a lot of pixel values in the negative range in that plot.

```python
# A histogram can also be helpful to look at the range of values in your data
# What do you notice about the histogram below?
dtm_pre_arr.plot.hist(color="purple")
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/177872252-d7b6f4fe-5085-4650-8a1b-0a9557a82075.png)

```
Histogram for your lidar DTM. Notice the number of values that are below 0. This suggests that there may be no data values in the data.
```

Looking at the min and max values of the data, you can see a very small negative number for the minimum. This number matches the nodata value that you looked at above.

```python
print("the minimum raster value is: ", np.nanmin(dtm_pre_arr.values))
print("the maximum raster value is: ", np.nanmax(dtm_pre_arr.values))
```

### Raster Data Exploration - Min and Max Values

Looking at the minimum value of the data, there is one of two things going on that need to be fixed:

- there may be no data values in the data with a negative value that are skewing your plot colors
- there also could be outlier data in your raster

You can explore the first option - that there are no data values by reading in the data and masking no data values using the **masked=True** parameter

Above you may have also noticed that the array has an additional dimension for the **“band”**. While the raster only has one layer - there is a 1 in the output of shape that could get in the way of processing.

You can remove that additional dimension using _.squeeze()_

```python
# Notice that the shape of this object has a 1 at the beginning that may caused an issue for plotting
dtm_pre_arr.shape

# Open the data and mask no data values
# Squeeze reduces the third dimension given there is only one "band" or layer to this data
dtm_pre_arr = rxr.open_rasterio(dem_pre_path, masked=True).squeeze()
# Notice there are now only 2 dimensions to your array
dtm_pre_arr.shape
```

Plot the data again to see what has changed. Now you have a reasonable range of data values and the data plot as you might expect it to.

```python
# Plot the data and notice that the scale bar looks better
# No data values are now masked
f, ax = plt.subplots(figsize=(10, 5))
dtm_pre_arr.plot(cmap="Greys_r",
                 ax=ax)
ax.set_title("Lidar Digital Elevation Model (DEM) \n Boulder Flood 2013")
ax.set_axis_off()
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/177873906-b6e4e629-c556-43ab-b9cb-c192189fd136.png)

The histogram has also changed. Now, it shows a reasonable distribution of pixel values.

```python
dtm_pre_arr.plot.hist(color="purple",
                      bins=20)
ax.set_title("Histogram of the Data with No Data Values Removed")
plt.show()
```

Notice that now the minimum value looks more like an elevation value (which should most often not be negative).

```python
print("The minimum raster value is: ", np.nanmin(dtm_pre_arr.data))
print("The maximum raster value is: ", np.nanmax(dtm_pre_arr.data))
```

![image](https://user-images.githubusercontent.com/43855029/177873975-7eb7f577-c46e-4a7e-ab86-f6993b825173.png)

### 3.3 Plot Raster and Vector data together

If you want, you can also add shapefile overlays to your raster data plot. Below you open a single shapefile using Geopandas that contains a boundary layer that you can overlay on top of your raster dataset

```python
# Open site boundary vector layer
site_bound_path = "data/colorado-flood/spatial/boulder-leehill-rd/clip-extent.shp"

site_bound_shp = gpd.read_file(site_bound_path)

# Plot the vector data
f, ax = plt.subplots(figsize=(8,4))
site_bound_shp.plot(color='teal',
                    edgecolor='black',
                    ax=ax)
ax.set(title="Site Boundary Layer - Shapefile")
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/177874542-a8e31efb-dadc-4714-91d5-9b4dff1d173c.png)

Once you have your shapefile open, can plot the two datasets together and begin to create a map.

```python
f, ax = plt.subplots(figsize=(11, 4))

dtm_pre_arr.plot.imshow(cmap="Greys",
                        ax=ax)
site_bound_shp.plot(color='None',
                    edgecolor='teal',
                    linewidth=2,
                    ax=ax,
                    zorder=4)

ax.set(title="Raster Layer with Vector Overlay")
ax.axis('off')
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/177874636-ee549b6b-a086-492c-8cc8-7734c946e8b6.png)

You now have the basic skills needed to open and plot raster data using **rioxarray** and **xarray**. In the following lessons, you will learn more about exploring and processing raster data.

### 3.4 Raster Metadata 

#### Coordinate Reference System (CRS)

The Coordinate Reference System or **CRS** of a spatial object tells Python where the **raster** is located in geographic space. It also tells Python what mathematical method should be used to **“flatten”** or project the raster in geographic space.

![image](https://user-images.githubusercontent.com/43855029/178023785-fa7dd55c-d1a1-4f3e-9bbb-7309bef05a52.png)
_Maps of the United States in different projections. Notice the differences in shape associated with each different projection. These differences are a direct result of the calculations used to "flatten" the data onto a 2-dimensional map. Source: M. Corey, opennews.org_

**Note**: data from the same location but saved in different coordinate references systems will not line up in any GIS or other program.

You can view the CRS using crs() method in Python

```python
# Import necessary packages
import os

import matplotlib.pyplot as plt
import rioxarray as rxr
import earthpy as et

# Get data and set working directory
et.data.get_data("colorado-flood")
os.chdir(os.path.join(et.io.HOME,
                      'earth-analytics',
                      'data'))
                      
# Define relative path to file
lidar_dem_path = "colorado-flood/spatial/boulder-leehill-rd/pre-flood/lidar/pre_DTM.tif"

# View crs of raster imported with rasterio
lidar_dem = rxr.open_rasterio(lidar_dem_path, masked=True)
print("The CRS of this data is:", lidar_dem.rio.crs)
```

The CRS object is 32613 which you can look up on the [spatial reference.org website](http://www.spatialreference.org/ref/epsg/32613/)

#### Raster Extent

The spatial extent of a raster or spatial object is the geographic area that the raster data covers.

![image](https://user-images.githubusercontent.com/43855029/178024857-68b14420-5df8-4495-9c5d-34544b6390d4.png)

```python
lidar_dem.rio.bounds()
```

#### Raster Resolution

A raster has horizontal (x and y) resolution. This resolution represents the area on the ground that each pixel covers. The units for your data are in meters as determined by the CRS above. In this case, your data resolution is 1 x 1. This means that each pixel represents a 1 x 1 meter area on the ground. You can view the resolution of your data using the .res function.

```python
lidar_dem.rio.resolution()
```

### 3.5 Raster processing

#### Subtracting Raster

**Canopy Height Model**: represents the HEIGHT of the trees. This is not an elevation value, rather it’s the height or distance between the ground and the top of the trees (or buildings or whatever object that the lidar system detected and recorded).

Some canopy height models also include buildings, so you need to look closely at your data to make sure it was properly cleaned before assuming it represents all trees!

**CHM = DSM - DEM**

Code to subtract 2 rasters:

```python
# Load DTM and DSM rasters:
pre_flood_path = "data/colorado-flood/spatial/boulder-leehill-rd/pre-flood/lidar/"
pre_DTM = rxr.open_rasterio(pre_flood_path+"pre_DTM.tif",masked=True).squeeze()
pre_DSM = rxr.open_rasterio(pre_flood_path+"pre_DSM.tif",masked=True).squeeze()

# Check if DTM and DSM have the same spatial extend and resolution?
print("Is the spatial extent the same?",
      pre_DTM.rio.bounds() == pre_DSM.rio.bounds())

# Is the resolution the same ??
print("Is the resolution the same?",
      pre_DTM.rio.resolution() == pre_DSM.rio.resolution())      
```

```python
# Calculate canopy height model
CHM = pre_DSM - pre_DTM

# Plot the data
f, ax = plt.subplots(figsize=(10, 5))
CHM.plot(cmap="Greens")
ax.set(title="Canopy Height Model for Lee Hill Road Pre-Flood")
ax.set_axis_off()
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/178037429-2fc284c2-e0ca-4cf0-8e06-70e532fa58f7.png)

Explore the histogram to see the range of raster values

```python
CHR.plot.hist()
plt.show()
```
![image](https://user-images.githubusercontent.com/43855029/178037579-d47cb52c-5bc4-4604-91ad-5b88e2f57c81.png)

Export CHR to geotiff format

```python
CHM.rio.to_raster(pre_flood_path+"CHR.tif")
```

Spatial plot with masked value above 5
```python
import xarray as xr
class_bins = [-np.inf, 2, 7, 12, np.inf]
CHM_class = xr.apply_ufunc(np.digitize,CHM,class_bins)

# Mask out values not equalt to 5
CHM_class_ma = CHM_class.where(CHM_class != 5)

im = CHM_class_ma.plot.imshow()
ax.set_axis_off()
```

![image](https://user-images.githubusercontent.com/43855029/178041608-1171d111-9f05-4dcb-a915-4c6ce9742ced.png)


Using legend:

```python
from matplotlib.colors import ListedColormap, BoundaryNorm

# Create a list of labels to use for your legend
height_class_labels = ["Short trees",
                       "Medium trees",
                       "Tall trees",
                       "Really tall trees"]

# Create a colormap from a list of colors
colors = ['linen',
          'lightgreen',
          'darkgreen',
          'maroon']

cmap = ListedColormap(colors)

class_bins = [.5, 1.5, 2.5, 3.5, 4.5]
norm = BoundaryNorm(class_bins,
                    len(colors))

# Plot newly classified and masked raster
f, ax = plt.subplots(figsize=(10, 5))
im = CHM_class_ma.plot.imshow(cmap=cmap,
                                        norm=norm,
                                        # Turn off colorbar
                                        add_colorbar=False)
# Add legend using earthpy
ep.draw_legend(im,
               titles=height_class_labels)
ax.set(title="Classified Lidar Canopy Height Model \n Derived from NEON AOP Data")
ax.set_axis_off()
plt.show()
```
![image](https://user-images.githubusercontent.com/43855029/178041562-1c315bf3-4faa-4850-9e83-66cfbc2f6048.png)


#### Clipping raster

We and clip raster using **clip()** function.
Here we clip the raster data using shapefile

Load the vector data:

```python
shp_path = "data/colorado-flood/spatial/boulder-leehill-rd/"

# Open crop extent (your study area extent boundary)
crop_extent = gpd.read_file(shp_path+"clip-extent.shp")
```

Impose the shapefile over to raster

```python
f, ax = plt.subplots(figsize=(10, 5))
CHM_class_ma.plot.imshow(ax=ax)

crop_extent.plot(ax=ax,
                 alpha=.8)
ax.set(title="Raster Layer with Shapefile Overlayed")

ax.set_axis_off()
plt.show()
```
![image](https://user-images.githubusercontent.com/43855029/178043550-9d6f5a22-c374-4cdd-a057-9f021dfb6b16.png)


Cliping

```python
from shapely.geometry import mapping

CHM_clipped = CHM_class_ma.rio.clip(crop_extent.geometry.apply(mapping),
                                      # This is needed if your GDF is in a diff CRS than the raster data
                                      crop_extent.crs)

f, ax = plt.subplots(figsize=(10, 4))
CHM_clipped.plot(ax=ax)
ax.set(title="Raster Layer Cropped to Geodataframe Extent")
ax.set_axis_off()
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/178043758-f6b9ac6e-430e-496b-a402-4caa2581c800.png)

### 3.6 Extract point data from raster

Load raster:

```python
sjer_lidar_chm_path = os.path.join("data",
                                   "spatial-vector-lidar",
                                   "california", 
                                   "neon-sjer-site",
                                   "2013", 
                                   "lidar")

sjer_chm = rxr.open_rasterio(sjer_lidar_chm_path+"/SJER_lidarCHM.tif", masked=True).squeeze()
```

Load shapefile:

```python
vector_path = os.path.join("data",
                           "spatial-vector-lidar",
                           "california", 
                           "neon-sjer-site",
                           "vector_data")

sjer_plots_points = gpd.read_file(vector_path+"/SJER_plot_centroids.shp")
```

Plotting overlay

```python
fig, ax = plt.subplots(figsize=(10, 10))

# We plot with the zeros in the data so the CHM can be better represented visually
ep.plot_bands(sjer_chm,
              extent=plotting_extent(sjer_chm,
                                     sjer_chm.rio.transform()),  # Set spatial extent
              cmap='Greys',
              title="San Joachin Field Site \n Vegetation Plot Locations",
              scale=False,
              ax=ax)

sjer_plots_points.plot(ax=ax,
                       marker='s',
                       #markersize=45,
                       color='purple')
ax.set_axis_off()
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/178046403-ebf3618f-b689-4ef8-a0b0-9d41e7ef32c5.png)

Clean up the 0 value

```python
# CLEANUP: Set CHM values of 0 to NAN (no data or not a number)
sjer_chm_data_no_zeros = sjer_chm_data.where(sjer_chm_data != 0, np.nan)
```

```python
# Create a buffered polygon layer from your plot location points
sjer_plots_poly = sjer_plots_points.copy()

# Buffer each point using a 20 meter circle radius
# and replace the point geometry with the new buffered geometry
sjer_plots_poly["geometry"] = sjer_plots_points.geometry.buffer(20)
sjer_plots_poly.head()
```

```python
plot_buffer_path = os.path.join("data",
                                   "spatial-vector-lidar",
                                   "california", 
                                   "neon-sjer-site",                               
                                   "plot_buffer.shp")

sjer_plots_poly.to_file(plot_buffer_path)
```

Extract

```python
import rasterstats as rs

# Extract zonal stats
sjer_tree_heights = rs.zonal_stats(plot_buffer_path,
                                   sjer_chm_no_zeros.values,
                                   nodata=-999,
                                   affine=sjer_chm_no_zeros.rio.transform(),
                                   geojson_out=True,
                                   copy_properties=True,
                                   stats="count min mean max median")

# Turn extracted data into a pandas geodataframe
sjer_lidar_height_df = gpd.GeoDataFrame.from_features(sjer_tree_heights)
sjer_lidar_height_df.head()
```

Plot

```python
fig, ax = plt.subplots(figsize=(10, 5))

ax.bar(sjer_lidar_height_df['Plot_ID'],
       sjer_lidar_height_df['max'],
       color="purple")

ax.set(xlabel='Plot ID', 
       ylabel='Max Height',
       title='Maximum LIDAR Derived Tree Heights')

plt.setp(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.show()
```

![image](https://user-images.githubusercontent.com/43855029/178048471-aabb4fcd-84c6-401a-a995-52e03580d6d4.png)


