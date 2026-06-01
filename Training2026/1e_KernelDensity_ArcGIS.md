# Kernel Density Estimation of Natural Cover Loss in Keo Seima

The goal of this training is to estimate the density of change across Keo Seima and visualize the hotspots at which this change occurs. This will be done in **three main sections:** 

#### 1) Estimating the Density of Change Across the change Period: This will involve calculating the overall density of all changes that occurred across the entire 7-year period.

#### 2) Visualizing the Shifts in Hotspots: This will involve creating biannual Kernel Density maps for individual change detection maps, providing insight into how hotspots have shifted over time.

#### 3) Visualizing Hotspots for Individual Land Cover Types: this will involve creating Kernel Density maps for specific land cover types (Forest, Open Forest, and Grassland) transition to croplands.

These three analyses will help us understand where the areas of greatest concern are, based on the density of change in a given area. Furthermore, the latter sections will help visualize temporal trends and highlight where specific land cover types are most vulnerable to change.

**Guide to Kernel Density:**

(https://pro.arcgis.com/en/pro-app/latest/tool-reference/spatial-analyst/how-kernel-density-works.htm)

<img width="1563" height="765" alt="image" src="https://github.com/user-attachments/assets/841c6dec-b6ad-45f3-833a-9ca9950f7094" />


<img width="1689" height="960" alt="image" src="https://github.com/user-attachments/assets/b67cfd22-4fdb-485b-a931-18af183a173b" />


#
# Step 1
Download and Open the **ArcGIS Project Package** or continue your work from the previous **1d_ChangeDetection** tutorial. 

# Step 2
Provided in **Post-Processed Land Cover** group in the contents pane are a raster and shapefile containing the change detection results. Symbolize the layers and open their attribute table to explore the data that they contain. 

<img width="2604" height="1366" alt="image" src="https://github.com/user-attachments/assets/304fc06d-88ea-431d-aac5-c7be2f500ded" />


# Step 3
>[!Note]
>Point (or line) features are required to be input into a Kernel Density model. This will require
>converting the raster layer representing deforestation or landcover change to a point layer.

Open the **Raster to Point** tool in the geoprocessing toolbox. 

Convert the raster cells to points, **based on the value field** (land cover class). 

This will create a new point feature layer where every cell in the raster dataset will be assigned a point. This may take some time to process and visualize. 

<img width="920" height="1565" alt="image" src="https://github.com/user-attachments/assets/b96b8546-aa0b-421c-ab13-d49874a690f7" />

<img width="1212" height="825" alt="image" src="https://github.com/user-attachments/assets/73e787f5-a49a-44c7-8baa-50a7472f74ed" />

# Step 4
In the Geoprocessing Toolbox, search for and select the **Kernel Density tool**.

>[!WARNING]
>This step will take a significant amount of time to process (several minutes or more). For the purpose of this training the output file is provided.  To see what the output looks like open the “KernelDensity_AllDeforestation” file. This shows the hotpots of change across the 9 years.

A)	**Input point or polyline features:** select the newly created point feature layer (i.e. Deforestation_2017to2024_Point)

B)	**Population field:** NONE

C)	**Output Raster:** provide a logical name (KernelDensity_AllDeforestation)

D)	**Output cell size:** 10

E)	**Search Radius:** leave default

F)	**Area Units:** Hectares

G)	**Output cell values:** Densities

H)	**Method:** Planar

I)	**Input barrier features:** KSWS_2023F_WGS84

# Step 5 (optional, if completing a multi-step analysis)
Open the point layer created during Step 3 **(Raster to Point Output)**. Use the **Select by Attribute** tool to select the deforestation modeled between 2020 and 2024. 

<img width="2059" height="1643" alt="image" src="https://github.com/user-attachments/assets/1538b68f-4398-4952-bf70-799ce465cea9" />

# Step 6
Repeat Step 4 **(Kernel Density)** using this new, more recent, output from Step 5. 

# Step 7 (optional)
Use the **Select by Attribute** tool to select each of the 'Class_from' value 8, then rerun the Kernel Density analysis from Step #4. This allows us to look at the loss of Open Forest (gridcode = 8) independnetly. This step can be rerun for 'Class_from' = 7 (Dense Forest) or 'Class_from' = 9 (Grasslands). 

<img width="1531" height="1532" alt="image" src="https://github.com/user-attachments/assets/00545943-04ec-496f-888b-ac39d94169e1" />



# Discussion Question

#### 1) How can we imagine applying a barrier weighted kernel density to the Keo Seima (or similar) analysis? 
<img width="1465" height="928" alt="image" src="https://github.com/user-attachments/assets/f5ed29d0-9018-4802-84e3-6ccd7e8af4df" />

#### 2) How could we use a **'Space Time Kernel Density'**?
([https://pro.arcgis.com/en/pro-app/3.4/tool-reference/spatial-analyst/space-time-kernel-density.htm](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/spatial-analyst/how-space-time-kernel-density-works.htm))

(https://www.esri.com/arcgis-blog/products/arcgis-pro/analytics/density-tools-in-arcgis-pro)

<img width="1419" height="832" alt="image" src="https://github.com/user-attachments/assets/b282bc55-6ce2-4c6d-b1bf-490b0f1268c3" />

### 3) Image how you could more dynamically track the types or causes of land cover conversion using the Emerging Hot Spot Analysis toolset.
<img width="1531" height="1532" alt="image" src="https://github.com/user-attachments/assets/11040f77-7bce-4654-8725-d91c120e1b8c" />

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)

