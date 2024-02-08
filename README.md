GEOG0051 Mining Social and Geographic Datasets
-----------------------------------

Formative Assessment (JSBD8)
-------------------------------

<<<<<<< Updated upstream
Note: Notebook might contain scripts and instructions adapted from GEOG0115, GEOG0051. 
Contributors: Stephen Law, Mateo Neira, Nikki Tanu, Thomas Keel, Gong Jie, Jason Tang and Demin Hu.


=======
Street Network Analysis to choose cycle lane locations
>>>>>>> Stashed changes
===============
> ### Target Area: Kensington


```python
# Import relevant libraries
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import osmnx as ox
import networkx as nx
import matplotlib.cm as cm
import matplotlib.colors as colors
```

Research question: Where cycle lanes should be introduced?

Method: First, the rental cycle spots are searched from OpenStreetMap. Second, assuming that the cyclists choose the shortest pass to travel, the travel routes are calculated for all the pairs of the rental cycle spots. Third, the most frequent edges are estimated, mapping the frequency difference by tints.

1) Loading the bike network graph of Kensington with a radius of 2000m.


```python
G = ox.graph_from_address('Kensington, UK', dist = 2000, network_type = 'bike')
ox.plot_graph(G, node_size = 2, node_color = 'w', node_alpha = 0.5)
```


    
![png](GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_files/GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_6_0.png)
    





    (<Figure size 800x800 with 1 Axes>, <Axes: >)



2) Loading the rental cycle spots. The radius is 1200m for taking buffer because a 2000m-radius map cannot estimate the shortest passes for all the pairs e.g., when the rental cycle spots located on the corners are not connected by the edges.


```python
# Getting the geometries of Westminster 
tags = {'amenity': True, 'highway':True, 'landuse':True, 'building':True, 'waterway': True, 'railway': True}
all_geom=ox.geometries.geometries_from_address('Kensington, UK', tags, dist=1200)
all_geom = all_geom.to_crs(epsg=3857) # The crs of 3857 can only be used to plot with contextily.
```

    C:\Users\Kazuhiro Shibayama\AppData\Local\Temp\ipykernel_10864\1326658294.py:3: UserWarning: The `geometries` module and `geometries_from_X` functions have been renamed the `features` module and `features_from_X` functions. Use these instead. The `geometries` module and function names are deprecated and will be removed in a future release.
      all_geom=ox.geometries.geometries_from_address('Kensington, UK', tags, dist=1200)
    


```python
fig,ax = plt.subplots(figsize=(10,10))
all_geom[all_geom['amenity'] == 'bicycle_rental'].plot(ax=ax,color='black')
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()
```


    
![png](GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_files/GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_9_0.png)
    


3) Dividing the geocodes of the rental cycle spots to latitude and longitude, and the nearest nodes are estimated.


```python
bicycle_rental = all_geom[all_geom['amenity'] == 'bicycle_rental']
bicycle_rental = bicycle_rental.to_crs(epsg=4326) # The crs of 4326 can be used to plot shortest-paths.
bicycle_rental
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>direction</th>
      <th>highway</th>
      <th>geometry</th>
      <th>surface</th>
      <th>traffic_calming</th>
      <th>name</th>
      <th>created_by</th>
      <th>ref:GB:tflcid</th>
      <th>note</th>
      <th>crossing</th>
      <th>...</th>
      <th>orientation</th>
      <th>taxi</th>
      <th>tower:type</th>
      <th>3dmr</th>
      <th>roof:colour</th>
      <th>telephone_kiosk</th>
      <th>ways</th>
      <th>castle_type</th>
      <th>name:lv</th>
      <th>building:cladding</th>
    </tr>
    <tr>
      <th>element_type</th>
      <th>osmid</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="47" valign="top">node</th>
      <th>823375879</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18261 51.49420)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>835975572</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18494 51.50162)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>De Vere Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>835975573</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18438 51.50201)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Palace Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>835975574</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18546 51.49544)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Emperor's Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>836785671</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19626 51.49079)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Trebovir Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>836785673</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18674 51.49158)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Collingham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>836785680</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19245 51.49156)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Penywern Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>836785688</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19053 51.49013)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Bramham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>836785695</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19061 51.49359)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Knaresborough Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>849584972</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19370 51.50851)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Palace Garden Terrace</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>849584974</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19256 51.50478)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Vicarage Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>849584979</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19899 51.50655)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Campden Hill Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>849584982</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19636 51.50935)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Notting Hill Gate Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>850057438</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19140 51.50303)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Church Street</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>862179889</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18383 51.49797)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road (North)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>885331201</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19765 51.49971)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Phillimore Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>885331209</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19301 51.50032)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Wright's Lane</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>885331222</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19728 51.49739)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Abingdon Villas</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>885331227</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20524 51.49668)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Warwick Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>885331230</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19543 51.50037)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Argyll Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>891044385</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20274 51.50068)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Ilchester Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>891044386</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20948 51.49812)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Olympia Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>892452179</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19838 51.49370)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>West Cromwell Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>892452182</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19478 51.49335)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Nevern Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>892452184</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19186 51.49585)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lexham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>892452189</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19157 51.50141)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Derry Street</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1057788707</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19433 51.50198)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Town Hall</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1176800575</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19228 51.49713)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Marloes Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1176868173</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20073 51.50235)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Holland Park</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1706931098</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20593 51.50137)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Abbotsbury Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2376356131</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21588 51.50371)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Hansard Mews</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3794078857</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19354 51.50825)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4623458572</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21097 51.50420)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Addison Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5053068176</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18329 51.49653)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road (Central)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5247920389</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21565 51.49425)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Brook Green South</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6204029685</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21601 51.50925)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Rifle Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6614676127</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18303 51.49487)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6708146798</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20850 51.50645)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Princedale Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6868062041</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21606 51.50652)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Queensdale Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6879017137</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21144 51.50003)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Russell Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914056390</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20596 51.49087)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>West Kensington Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914056393</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20938 51.48960)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Vereker Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914056395</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20467 51.50960)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lansdowne Walk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914066085</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20918 51.49101)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gwendwr Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914066285</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21509 51.49020)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Barons Court Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6914066385</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20554 51.50750)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lansdowne Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9622712338</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21347 51.49693)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Blythe Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>47 rows × 505 columns</p>
</div>




```python
bicycle_rental['latitude']  = bicycle_rental['geometry'].y
bicycle_rental['longitude'] = bicycle_rental['geometry'].x
bicycle_rental
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>direction</th>
      <th>highway</th>
      <th>geometry</th>
      <th>surface</th>
      <th>traffic_calming</th>
      <th>name</th>
      <th>created_by</th>
      <th>ref:GB:tflcid</th>
      <th>note</th>
      <th>crossing</th>
      <th>...</th>
      <th>tower:type</th>
      <th>3dmr</th>
      <th>roof:colour</th>
      <th>telephone_kiosk</th>
      <th>ways</th>
      <th>castle_type</th>
      <th>name:lv</th>
      <th>building:cladding</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
    <tr>
      <th>element_type</th>
      <th>osmid</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="47" valign="top">node</th>
      <th>823375879</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18261 51.49420)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.494196</td>
      <td>-0.182609</td>
    </tr>
    <tr>
      <th>835975572</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18494 51.50162)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>De Vere Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.501618</td>
      <td>-0.184936</td>
    </tr>
    <tr>
      <th>835975573</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18438 51.50201)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Palace Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.502011</td>
      <td>-0.184379</td>
    </tr>
    <tr>
      <th>835975574</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18546 51.49544)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Emperor's Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.495440</td>
      <td>-0.185465</td>
    </tr>
    <tr>
      <th>836785671</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19626 51.49079)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Trebovir Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.490792</td>
      <td>-0.196261</td>
    </tr>
    <tr>
      <th>836785673</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18674 51.49158)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Collingham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.491578</td>
      <td>-0.186739</td>
    </tr>
    <tr>
      <th>836785680</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19245 51.49156)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Penywern Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.491556</td>
      <td>-0.192454</td>
    </tr>
    <tr>
      <th>836785688</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19053 51.49013)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Bramham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.490135</td>
      <td>-0.190529</td>
    </tr>
    <tr>
      <th>836785695</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19061 51.49359)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Knaresborough Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.493592</td>
      <td>-0.190609</td>
    </tr>
    <tr>
      <th>849584972</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19370 51.50851)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Palace Garden Terrace</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.508511</td>
      <td>-0.193698</td>
    </tr>
    <tr>
      <th>849584974</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19256 51.50478)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Vicarage Gate</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.504785</td>
      <td>-0.192558</td>
    </tr>
    <tr>
      <th>849584979</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19899 51.50655)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Campden Hill Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.506555</td>
      <td>-0.198991</td>
    </tr>
    <tr>
      <th>849584982</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19636 51.50935)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Notting Hill Gate Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.509353</td>
      <td>-0.196357</td>
    </tr>
    <tr>
      <th>850057438</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19140 51.50303)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Church Street</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.503028</td>
      <td>-0.191404</td>
    </tr>
    <tr>
      <th>862179889</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18383 51.49797)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road (North)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.497973</td>
      <td>-0.183834</td>
    </tr>
    <tr>
      <th>885331201</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19765 51.49971)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Phillimore Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.499710</td>
      <td>-0.197650</td>
    </tr>
    <tr>
      <th>885331209</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19301 51.50032)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Wright's Lane</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.500315</td>
      <td>-0.193006</td>
    </tr>
    <tr>
      <th>885331222</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19728 51.49739)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Abingdon Villas</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.497394</td>
      <td>-0.197277</td>
    </tr>
    <tr>
      <th>885331227</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20524 51.49668)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Warwick Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.496683</td>
      <td>-0.205243</td>
    </tr>
    <tr>
      <th>885331230</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19543 51.50037)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Argyll Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.500371</td>
      <td>-0.195434</td>
    </tr>
    <tr>
      <th>891044385</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20274 51.50068)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Ilchester Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.500684</td>
      <td>-0.202742</td>
    </tr>
    <tr>
      <th>891044386</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20948 51.49812)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Olympia Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.498121</td>
      <td>-0.209478</td>
    </tr>
    <tr>
      <th>892452179</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19838 51.49370)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>West Cromwell Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.493702</td>
      <td>-0.198382</td>
    </tr>
    <tr>
      <th>892452182</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19478 51.49335)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Nevern Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.493352</td>
      <td>-0.194776</td>
    </tr>
    <tr>
      <th>892452184</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19186 51.49585)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lexham Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.495847</td>
      <td>-0.191859</td>
    </tr>
    <tr>
      <th>892452189</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19157 51.50141)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Derry Street</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.501409</td>
      <td>-0.191565</td>
    </tr>
    <tr>
      <th>1057788707</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19433 51.50198)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Kensington Town Hall</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.501978</td>
      <td>-0.194334</td>
    </tr>
    <tr>
      <th>1176800575</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19228 51.49713)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Marloes Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.497132</td>
      <td>-0.192277</td>
    </tr>
    <tr>
      <th>1176868173</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20073 51.50235)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Holland Park</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.502349</td>
      <td>-0.200731</td>
    </tr>
    <tr>
      <th>1706931098</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20593 51.50137)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Abbotsbury Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.501371</td>
      <td>-0.205932</td>
    </tr>
    <tr>
      <th>2376356131</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21588 51.50371)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Hansard Mews</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.503706</td>
      <td>-0.215883</td>
    </tr>
    <tr>
      <th>3794078857</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.19354 51.50825)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.508247</td>
      <td>-0.193538</td>
    </tr>
    <tr>
      <th>4623458572</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21097 51.50420)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Addison Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.504200</td>
      <td>-0.210974</td>
    </tr>
    <tr>
      <th>5053068176</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18329 51.49653)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gloucester Road (Central)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.496529</td>
      <td>-0.183292</td>
    </tr>
    <tr>
      <th>5247920389</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21565 51.49425)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Brook Green South</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.494250</td>
      <td>-0.215652</td>
    </tr>
    <tr>
      <th>6204029685</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21601 51.50925)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Rifle Place</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.509248</td>
      <td>-0.216008</td>
    </tr>
    <tr>
      <th>6614676127</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.18303 51.49487)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.494870</td>
      <td>-0.183031</td>
    </tr>
    <tr>
      <th>6708146798</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20850 51.50645)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Princedale Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.506455</td>
      <td>-0.208501</td>
    </tr>
    <tr>
      <th>6868062041</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21606 51.50652)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Queensdale Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.506522</td>
      <td>-0.216063</td>
    </tr>
    <tr>
      <th>6879017137</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21144 51.50003)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Russell Gardens</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.500028</td>
      <td>-0.211440</td>
    </tr>
    <tr>
      <th>6914056390</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20596 51.49087)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>West Kensington Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.490872</td>
      <td>-0.205957</td>
    </tr>
    <tr>
      <th>6914056393</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20938 51.48960)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Vereker Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.489600</td>
      <td>-0.209379</td>
    </tr>
    <tr>
      <th>6914056395</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20467 51.50960)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lansdowne Walk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.509600</td>
      <td>-0.204666</td>
    </tr>
    <tr>
      <th>6914066085</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20918 51.49101)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gwendwr Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.491011</td>
      <td>-0.209183</td>
    </tr>
    <tr>
      <th>6914066285</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21509 51.49020)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Barons Court Station</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.490200</td>
      <td>-0.215087</td>
    </tr>
    <tr>
      <th>6914066385</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.20554 51.50750)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lansdowne Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.507500</td>
      <td>-0.205536</td>
    </tr>
    <tr>
      <th>9622712338</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>POINT (-0.21347 51.49693)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Blythe Road</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>51.496933</td>
      <td>-0.213472</td>
    </tr>
  </tbody>
</table>
<p>47 rows × 507 columns</p>
</div>




```python
bicycle_rental_nearest = ox.nearest_nodes(G,bicycle_rental['longitude'],bicycle_rental['latitude'])
bicycle_rental_nearest
```




    [26389004,
     26389506,
     862464696,
     78530684,
     7330291114,
     57650629,
     32585953,
     57650631,
     1288714197,
     102093,
     7538491623,
     7456829135,
     25579302,
     10074518829,
     276552,
     25502538,
     26389642,
     26389663,
     885331207,
     973649552,
     255469345,
     1617991936,
     109886,
     197982,
     76611556,
     11776200,
     11776198,
     78560432,
     5515430225,
     7416302958,
     665991140,
     7549199020,
     25481320,
     1265917665,
     9161067287,
     6204029281,
     5898890847,
     216162,
     8654631176,
     6363688696,
     33206587,
     20961337,
     7342435909,
     33203952,
     20960971,
     94239831,
     1414732229]



4) Calculating the shortest routes for all the pairs of the rental cycle spots.


```python
import itertools
```


```python
origs = bicycle_rental_nearest
dests = bicycle_rental_nearest


paths = []
for o, d in itertools.product(origs,dests):
    path = nx.shortest_path(G, o, d, weight='length' )
    paths.append(path)
    
len(paths) #100
```




    2209




```python
# get the edges as Geodataframe
gdf_nodes, gdf_edges = ox.graph_to_gdfs(G)

```

5) Loading the edge information from the OpenStreetMap, and counting how often each edge appears in the shortest passes calculated above.


```python
gdf_edges
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>osmid</th>
      <th>lanes</th>
      <th>name</th>
      <th>highway</th>
      <th>maxspeed</th>
      <th>oneway</th>
      <th>reversed</th>
      <th>length</th>
      <th>geometry</th>
      <th>ref</th>
      <th>bridge</th>
      <th>access</th>
      <th>tunnel</th>
      <th>junction</th>
      <th>service</th>
      <th>width</th>
      <th>est_width</th>
    </tr>
    <tr>
      <th>u</th>
      <th>v</th>
      <th>key</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">102061</th>
      <th>1280521211</th>
      <th>0</th>
      <td>4486923</td>
      <td>2</td>
      <td>Strathearn Place</td>
      <td>tertiary</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>19.172</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17127 51.51360)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30522449</th>
      <th>0</th>
      <td>4486921</td>
      <td>NaN</td>
      <td>Stanhope Terrace</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>66.831</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17216 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30522451</th>
      <th>0</th>
      <td>4774389</td>
      <td>NaN</td>
      <td>Sussex Place</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>60.192</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17107 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2766801691</th>
      <th>0</th>
      <td>55969730</td>
      <td>2</td>
      <td>Sussex Place</td>
      <td>tertiary</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>33.976</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17172 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>102062</th>
      <th>2769935537</th>
      <th>0</th>
      <td>111575701</td>
      <td>NaN</td>
      <td>Stanhope Terrace</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>True</td>
      <td>False</td>
      <td>12.109</td>
      <td>LINESTRING (-0.17248 51.51305, -0.17253 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>11591015888</th>
      <th>7806099773</th>
      <th>0</th>
      <td>[1246779690, 1246779691, 1246779692]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>True</td>
      <td>34.006</td>
      <td>LINESTRING (-0.19141 51.51091, -0.19158 51.510...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>building_passage</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">11592569474</th>
      <th>33140924</th>
      <th>0</th>
      <td>4992854</td>
      <td>2</td>
      <td>Seagrave Road</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>13.868</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19664 51.48635)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11592569475</th>
      <th>0</th>
      <td>1246957982</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>False</td>
      <td>9.594</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19666 51.48620)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7746604922</th>
      <th>0</th>
      <td>4992854</td>
      <td>2</td>
      <td>Seagrave Road</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>56.040</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19646 51.486...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11592569475</th>
      <th>11592569474</th>
      <th>0</th>
      <td>1246957982</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>True</td>
      <td>9.594</td>
      <td>LINESTRING (-0.19666 51.48620, -0.19654 51.48625)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>9977 rows × 17 columns</p>
</div>




```python
# create the 'count' column in gdf_edges as default 0
gdf_edges['count'] = 0

# Temporary dictionary to keep track of edge counts
edge_counts_dict = {}

# Iterate through each path to count edge appearances
for path in paths:
    for start, end in zip(path[:-1], path[1:]):
        key = 0
        # Increment the count for this edge in the dictionary
        edge_counts_dict[(start, end, key)] = edge_counts_dict.get((start, end, key), 0) + 1

# Update the 'count' column in gdf_edges with the counts
for index, row in gdf_edges.iterrows():
    u, v, key = index
    gdf_edges.at[(u, v, key), 'count'] = edge_counts_dict.get((u, v, key), 0)
```


```python
# gdf_edges now contains the updated counts in the 'count' column
gdf_edges
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>osmid</th>
      <th>lanes</th>
      <th>name</th>
      <th>highway</th>
      <th>maxspeed</th>
      <th>oneway</th>
      <th>reversed</th>
      <th>length</th>
      <th>geometry</th>
      <th>ref</th>
      <th>bridge</th>
      <th>access</th>
      <th>tunnel</th>
      <th>junction</th>
      <th>service</th>
      <th>width</th>
      <th>est_width</th>
      <th>count</th>
    </tr>
    <tr>
      <th>u</th>
      <th>v</th>
      <th>key</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">102061</th>
      <th>1280521211</th>
      <th>0</th>
      <td>4486923</td>
      <td>2</td>
      <td>Strathearn Place</td>
      <td>tertiary</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>19.172</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17127 51.51360)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30522449</th>
      <th>0</th>
      <td>4486921</td>
      <td>NaN</td>
      <td>Stanhope Terrace</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>66.831</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17216 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30522451</th>
      <th>0</th>
      <td>4774389</td>
      <td>NaN</td>
      <td>Sussex Place</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>60.192</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17107 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2766801691</th>
      <th>0</th>
      <td>55969730</td>
      <td>2</td>
      <td>Sussex Place</td>
      <td>tertiary</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>33.976</td>
      <td>LINESTRING (-0.17155 51.51358, -0.17172 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>102062</th>
      <th>2769935537</th>
      <th>0</th>
      <td>111575701</td>
      <td>NaN</td>
      <td>Stanhope Terrace</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>True</td>
      <td>False</td>
      <td>12.109</td>
      <td>LINESTRING (-0.17248 51.51305, -0.17253 51.513...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>11591015888</th>
      <th>7806099773</th>
      <th>0</th>
      <td>[1246779690, 1246779691, 1246779692]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>True</td>
      <td>34.006</td>
      <td>LINESTRING (-0.19141 51.51091, -0.19158 51.510...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>building_passage</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">11592569474</th>
      <th>33140924</th>
      <th>0</th>
      <td>4992854</td>
      <td>2</td>
      <td>Seagrave Road</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>True</td>
      <td>13.868</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19664 51.48635)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11592569475</th>
      <th>0</th>
      <td>1246957982</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>False</td>
      <td>9.594</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19666 51.48620)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7746604922</th>
      <th>0</th>
      <td>4992854</td>
      <td>2</td>
      <td>Seagrave Road</td>
      <td>residential</td>
      <td>20 mph</td>
      <td>False</td>
      <td>False</td>
      <td>56.040</td>
      <td>LINESTRING (-0.19654 51.48625, -0.19646 51.486...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11592569475</th>
      <th>11592569474</th>
      <th>0</th>
      <td>1246957982</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>service</td>
      <td>NaN</td>
      <td>False</td>
      <td>True</td>
      <td>9.594</td>
      <td>LINESTRING (-0.19666 51.48620, -0.19654 51.48625)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>9977 rows × 18 columns</p>
</div>




```python
gdf_edges.sort_values(['count'], ascending=False).head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>osmid</th>
      <th>lanes</th>
      <th>name</th>
      <th>highway</th>
      <th>maxspeed</th>
      <th>oneway</th>
      <th>reversed</th>
      <th>length</th>
      <th>geometry</th>
      <th>ref</th>
      <th>bridge</th>
      <th>access</th>
      <th>tunnel</th>
      <th>junction</th>
      <th>service</th>
      <th>width</th>
      <th>est_width</th>
      <th>count</th>
    </tr>
    <tr>
      <th>u</th>
      <th>v</th>
      <th>key</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20960098</th>
      <th>7706821119</th>
      <th>0</th>
      <td>939548796</td>
      <td>4</td>
      <td>Earl's Court Road</td>
      <td>trunk</td>
      <td>30 mph</td>
      <td>True</td>
      <td>False</td>
      <td>3.330</td>
      <td>LINESTRING (-0.19611 51.49533, -0.19609 51.49530)</td>
      <td>A3220</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>266</td>
    </tr>
    <tr>
      <th>5898890864</th>
      <th>5898890866</th>
      <th>0</th>
      <td>[821523180, 821523181]</td>
      <td>3</td>
      <td>Cromwell Road</td>
      <td>trunk</td>
      <td>30 mph</td>
      <td>True</td>
      <td>False</td>
      <td>68.472</td>
      <td>LINESTRING (-0.19112 51.49470, -0.19143 51.494...</td>
      <td>A4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>261</td>
    </tr>
    <tr>
      <th>2808877323</th>
      <th>20960098</th>
      <th>0</th>
      <td>[307128372, 939548797]</td>
      <td>4</td>
      <td>Earl's Court Road</td>
      <td>trunk</td>
      <td>30 mph</td>
      <td>True</td>
      <td>False</td>
      <td>46.744</td>
      <td>LINESTRING (-0.19646 51.49569, -0.19625 51.495...</td>
      <td>A3220</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>249</td>
    </tr>
    <tr>
      <th>7721933067</th>
      <th>7715719502</th>
      <th>0</th>
      <td>307128373</td>
      <td>4</td>
      <td>Earl's Court Road</td>
      <td>trunk</td>
      <td>30 mph</td>
      <td>True</td>
      <td>False</td>
      <td>13.136</td>
      <td>LINESTRING (-0.19587 51.49505, -0.19578 51.49495)</td>
      <td>A3220</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>228</td>
    </tr>
    <tr>
      <th>7706821119</th>
      <th>7721933067</th>
      <th>0</th>
      <td>307128373</td>
      <td>4</td>
      <td>Earl's Court Road</td>
      <td>trunk</td>
      <td>30 mph</td>
      <td>True</td>
      <td>False</td>
      <td>31.756</td>
      <td>LINESTRING (-0.19609 51.49530, -0.19602 51.495...</td>
      <td>A3220</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>228</td>
    </tr>
  </tbody>
</table>
</div>



6) Visualizing the frequency of each road on the map.


```python
# set crs to 3857 (needed for contextily)
gdf_edges = gdf_edges.to_crs(epsg=3857) # setting crs to 3857
# plot edges according to degree centrality
ax=gdf_edges.plot('count',cmap='cool',figsize=(10,10))

# Customize legend
sm = plt.cm.ScalarMappable(cmap='cool')
sm._A = []  # empty array to indicate scalar mappable
cbar = plt.colorbar(sm, ax=ax)
cbar.set_label('Count')  # Set label for colorbar

# add a basemap using contextilly
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()
```


    
![png](GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_files/GEOG0051_Formative%20Asswssment%20%28For%20submission%29%20-%20%E3%82%B3%E3%83%94%E3%83%BC%20-%20%E3%82%B3%E3%83%94%E3%83%BC_24_0.png)
    


The thicker blue shows the roads more frequently likely to be used by cyclists. Because there is no cycle lane through Kensington High Street and Cromwell Road, which are coloured in thicker blues, when the local authorities consider the construction of cycle lanes, these two streets should be prioritized to benefit the most from the construction.

 Except, there are some limitations: First, the analysis is limited to Kensington, not considering the rental cycle spots out of the map. Second, this analysis does not consider cyclists using their private bicycles. Third, there are suitable travel distances for bicycle travel e.g., in that too short travel should be on foot, which is not considered. These missing viewpoints can bias the results. (294 words)
