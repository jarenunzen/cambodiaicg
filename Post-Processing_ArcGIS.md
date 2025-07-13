## The following is a guide for Post-Processing of the Google Earth Engine land cover products. This tutorial should be completed before conducting the Deforestation Analysis

Post Processing of the land cover maps (all 9: 2016-2024) is used here to create both Raster and Vector products cklipped to the users' desired spatial extent which can be used for further analysis. A secondary goal of this guide is to include basic cleaning and review of the maps which were generated during the GEE analysis. All pixel-based classifications contribute some amount of noise to the final product. We will use some ArcGIS tools to mitigate this noise. 

# Step 1   
Import Data for this training including:
- Boundary file for Keo Seima (.shp)
- Raster land cover maps from the GEE tutorial (.tif)

# Step 2
Use the **Raster to Polygon Tool** to convert each of the outputs from Google Earth Engine to polygon (vector) datasets. 
> This step must be run on: 2016.tif, 2017.tif, 2018.tif, 2019.tif, 2020.tif, 2021.tif, 2022.tif, 2023.tif, 2024.tif

> Procesing time per layer: ##minutes

>[!TIP]
>If this step takes a long time to process each layer, you can also choose to clip the data to the Keo Seima boundary before performing this conversion. Either the **Extract by Mask** or **RAster Clip** tools will work for this task, as long as you clip to the vector layer boundary.

<img width="760" height="1514" alt="image" src="https://github.com/user-attachments/assets/bf05bdb7-2751-454b-a01c-0e86a088490e" />

# Step 3
