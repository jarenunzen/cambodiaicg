## The following is a guide for Post-Processing of the Google Earth Engine land cover products. This tutorial should be completed before conducting the Deforestation Analysis

Post Processing of the land cover maps (all 9: 2016-2024) is used here to create both Raster and Vector products cklipped to the users' desired spatial extent which can be used for further analysis. A secondary goal of this guide is to include basic cleaning and review of the maps which were generated during the GEE analysis. All pixel-based classifications contribute some amount of noise to the final product. We will use some ArcGIS tools to mitigate this noise. 

# Step 1   
Import Data for this training including:
- Boundary file for Keo Seima (.shp)
- Raster land cover maps from the GEE tutorial (.tif)

# Step 2

