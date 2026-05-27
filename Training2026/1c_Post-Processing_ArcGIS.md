## The following is a guide for Post-Processing of the Google Earth Engine land cover products. This tutorial should be completed before conducting the Deforestation Analysis

Post Processing of the land cover maps (all 9: 2016-2024) is used here to create both Raster and Vector products cklipped to the users' desired spatial extent which can be used for further analysis. A secondary goal of this guide is to include basic cleaning and review of the maps which were generated during the GEE analysis. All pixel-based classifications contribute some amount of noise to the final product. We will use some ArcGIS tools to mitigate this noise. 

# Step 1   
Import Data for this training including:
- Boundary file for Keo Seima (.shp)
- Raster land cover maps from the GEE tutorial (.tif)

<img width="1956" height="1374" alt="image" src="https://github.com/user-attachments/assets/9b75fedc-e72d-4a36-a475-e17abbd0cdb0" />


# Step 2
Use the **Raster to Polygon Tool** to convert each of the outputs from Google Earth Engine to polygon (vector) datasets. 
> This step must be run on: 2016.tif, 2017.tif, 2018.tif, 2019.tif, 2020.tif, 2021.tif, 2022.tif, 2023.tif, 2024.tif

> Procesing time per layer: ##minutes

-**Input:** 2024 land cover raster output from GEE

-**Output:** KeoSeima_LandCover2024

-**Field:** Value [integer value for land cover class]

-**Simplify Polygons:** NO

-**Create Multipart Feature:** NO


>[!TIP]
>If this step takes a long time to process each layer, you can also choose to clip the data to the Keo Seima boundary before performing this conversion. Either the **Extract by Mask** or **Raster Clip** tools will work for this task, as long as you clip to the vector layer boundary.

<img width="760" height="1514" alt="image" src="https://github.com/user-attachments/assets/bf05bdb7-2751-454b-a01c-0e86a088490e" />

# Step 3
**Clip** the new vector output from Step #2 to the Keo Seima boundary layer.
> This step must be run on: 2016.shp, 2017.shp, 2018.shp, 2019.shp, 2020.shp, 2021.shp, 2022.shp, 2023.shp, 2024.shp

<img width="959" height="562" alt="image" src="https://github.com/user-attachments/assets/21c5641c-85ca-4dbc-9c14-854b56c9a328" />


# Step 4
Create a new Field using the attribute table. Name this field **"hectares"** with a TYPE of **float**. 

<img width="1704" height="650" alt="image" src="https://github.com/user-attachments/assets/5f13233b-7cba-4e86-a466-32d41fda9f3e" />


>[!WARNING]
>Using any other number (integer) or text type will not allow you to calculate polygon-specific sizes with decimal point values. 

# Step 5
In the attribute table **Calculate the Geometry** of the new attribute (field) using the Geodesic area and **hectares** unit. 

>[!TIP]
>Calculating the geoemtry of several hundred thousand polygons may take several minute or longer. This processing, which is necessary for each year, can be conducted simultaneously with Geoprocessing tools though. To save times, users are encouraged to intermix attribute-based calculations and geoprocessing tools for the remainder of this tutorial.

# Step 6
Once the area of all polygons within a layer has been calculated, use the **Select By Attributes** tool to set the Expression:

**hectares is less than or equal to 0.03**

<img width="2004" height="1107" alt="image" src="https://github.com/user-attachments/assets/cfba3f81-c730-4226-995a-b32597a525f6" />

# Step 7
Use the **Eliminate** tool to dissolve the selected polygons into their surrounding neighbors. 
>[!NOTE]
>Specifically, in this step we are specifying that polygons which are less than or equal to
>0.03 hecatres, not of the class of 'Developed' (gridcode = 6) merged with neighbors of the largest >size.
>The 0.03 hectares size threshold was used to eliminate polygons 3 cells in size or less. If your
>project has a specific Minimum Mapping Unit (MMU) this may be a more appropriate size to use.

<img width="772" height="1496" alt="image" src="https://github.com/user-attachments/assets/06727b63-232d-4e3d-85c0-88aafb08de78" />

# Step 8
**Dissolve** the newly revised polygon layers based on the gridcode field (land cover class integer attribute). At this point, we will want to have the **Create multipart features** option **ENABLED**.

>[!NOTE]
>(Optionally) users can choose to **SUM** the hectares/area field. Note that we will calculate a final area estimate for each class in the next step using the dissolved features. This value may differ and it is recomended to use that estimate instead of the one provided here. 

These vector layers will serve as the final vector land cover maps for this analysis. 

<img width="764" height="1492" alt="image" src="https://github.com/user-attachments/assets/6e2a28d9-d30a-4524-b6de-c121c3f7cf07" />

# Step 9
Repeat Steps #4 and #5 above, creating a new **'hectares'** field. This field will serve as the final estimate of land area per year and will be essential for subsequent analyses and reports. 

# Step 10
Convert each of the finalized land cover polygon (.shp) layers to Raster layers using the **Polygon to Raster** tool. These will be used for the Deforestation and Kernel Density analyses. 

<img width="763" height="1511" alt="image" src="https://github.com/user-attachments/assets/03c8dfda-a1c5-47f3-bcd7-e2337a383402" />

# Step 11 (optional)
As desired, provide symbology for the land cover **Polygon** and **Raster** layers. 

>[!TIP]
>The **Import Symbology** OR **Import Symbolgy from Layer** menu option (found in the upper-right of the Symbology pane) can be used to apply the symbology from one raster opr vector layer to another. This can save considerable time within this menu when creating symbology for multiple years.

<img width="864" height="1561" alt="image" src="https://github.com/user-attachments/assets/1232eb34-4cfa-4e5d-94d5-abe843fb9b68" />
