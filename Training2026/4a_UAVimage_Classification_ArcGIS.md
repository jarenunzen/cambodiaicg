# Header (change this to a graphical abstract and description)

**Objective:** Perform a supervised classification of UAV orthoimagery in ArcGIS Pro (v3.6 or v3.7) based on the 'forest', 'fallow', and 'cropland' classes. Additional information can be found in the supplemental Dropbox folder / UAV Resources

**Required Data:**
- UAV multispectral raster image (on Dropbox)
- Reference data shapefile

OR
- ArcGIS Pro Project Package

**Site:** The three communities of Andong Kralong, Sre Lvi, and Pu Kong, located in Mondulkiri province. WCS x GCBC Project Site
<img width="1280" height="612" alt="image002" src="https://github.com/user-attachments/assets/12769687-6584-4f68-a26e-160d546e8201" />

**For more information on this project, visit:** 

https://www.gcbc.org.uk/project/recognising-and-rewarding-the-contribution-of-indigenous-knowledge-for-the-sustainable-management-of-biodiversity/


## Part I: Preparing the data
### Step 1
Open the properties for the multispectral UAV raster image and explore the primary attributes such as the spatial resolition, radiometric resolution, number of bands, and band metadata.
- **Cell size:** 10.3cm
- **Radiometric resolution:** 16-bit unsigned
- **Number of bands:** 4
- **Band names:** 1- Green, 2- Red, 3- RedEdge, 4- Near Infrared (NIR)

  
<img width="1520" height="868" alt="image" src="https://github.com/user-attachments/assets/103e5738-cde4-4e22-a377-65cf6a1a0203" />


### Step 2
Select the image in the Table of Contents and navigate to 'Imagery' > 'Raster Functions' > 'Data Management' > 'Extract Bands'.

OR

Use the Add Data button to navigate to the UAV multispectral orthomosaic file. **Double-click** on the .tif file to see the individual bands ad add the 4 named image bands independently. 

<img width="1226" height="785" alt="image" src="https://github.com/user-attachments/assets/1711fba0-c325-4b61-8968-61361efc418c" />


>[!NOTE]
>This specific multispectral sensor does not include a standalone Blue band. For more information on this sensor, visit:https://enterprise.dji.com/mavic-3-m

### Step 3
Now we will build several several spectral indices, to be added to the multispectral UAV image, in aid of the classification. 


Navigate to the 'Imagery' tab, open 'Raster Functions' > 'Math' > 'Raster Calculator'. 
<img width="581" height="1161" alt="image" src="https://github.com/user-attachments/assets/18886417-50da-484c-85c6-858f9065b0be" />


>[!TIP]
>If you are working with a single, multi-band, raster image, then the Band Arithmetic tool can be used instead to specify discrete bands and custom equations.
>
><img width="131" height="120" alt="image" src="https://github.com/user-attachments/assets/ce842ec4-4cf7-4aff-b4f7-d503eec64f44" />


### Step 4
Select the NIR band and enter the **Raster Variable** 'n'. Select the red band and enter the **Raster Variable** 'r'.

Specify the **Expression** for the Normalized Difference Vegetation Index (NDVI) and click **Create new layer** at the bottom of the pane.

> (n - r) / (n + r)

<img width="581" height="1158" alt="image" src="https://github.com/user-attachments/assets/db9a390b-df1e-4bfa-9b7b-c9be915b4556" />


>[!CAUTION]
> This new (temporary) layer is created almost instantly. This layer and subsequent layers will only be temporarily saved within the project. If you close the project and re-open it, even if you save the project, this layer may be lost. To avoid losing this progress, we will ensure that the final composite of all of the layers are saved. **If you wish to save these intermediate indices as distinct layers, select Save as instead of Create new layer**.


### Step 5
Rename this new layer in the Table of Contents to **NDVI**. 

### Step 6
Repeat steps 3-5 for the Green-Red Vegetation Index (GRVI), Red-Green ration (RGI), and Anthocyanin Reflectance Index (ARI).

> GRVI = (g - r) / (g + r)
>
> RGI = (r/g)
>
> ARI = (1/g) - (1/re)

<img width="583" height="1163" alt="image" src="https://github.com/user-attachments/assets/57107962-b0b2-4e0c-ba8e-0042af6c18bd" />


### Step 7
Now we will calculate several texture metrics based on Haralick et al. 1973 (GLCM). 


Select the **NIR** band i nthe Table of Contents. 


Navigate to the Imagery tab > Raster Functions > Statistical > **Focal Statistics**. 


Select NIR as the Raster, in the neighborhood settings set both the Width and Height to 5, and choose **Mean** as the Statistics Type. 


Create a new layer. 

<img width="919" height="768" alt="image" src="https://github.com/user-attachments/assets/ab3ddd82-3408-471b-8c77-8dbf15429888" />


### Step 8
Repeat Step #7 to create new layers based on the following Statistics Types. 
- Median
- Standard Deviation
- Range
- 90th Percentile


<img width="250" height="174" alt="image" src="https://github.com/user-attachments/assets/c0099976-de75-4fc3-a4b1-14608d9c5527" />


>[!NOTE]
>In this tutorial, a standard 5x5 rectangular window is used to create texture metrics. This is not univerally the best option. Users may wish to adjust the window sizes based on the type of imagery (spatial resolution) or based on literature specific to their image type or features of interest (for example tree crowns vs croplands).


### Step 9
Search for the **Principal Compnents** tool in the Geoprocessing toolbox. 


Select the original UAV sensor bands (G, R, RE, NIR).


Enter **3** as the Number of prinicpal components and click **Run**. This step may take 6-10 minutes to process. 


<img width="579" height="484" alt="image" src="https://github.com/user-attachments/assets/c1db1e7d-dc7c-4bc8-bb25-f7966ffafbfa" />


>[!NOTE]
> Learn more about PCA from ESRI at: https://pro.arcgis.com/en/pro-app/3.4/tool-reference/spatial-analyst/how-principal-components-works.htm

### Step 10
Search for the **Composite Bands** tool within the Geoprocessing toolbox. 

Add each of the extracted bands, band indices, texture bands, and the PCA bands as the input raster layers. There should be 16 total bands (including the 3 from PCA). 

Ensure that the output raster name is descriptive and distinguishable. 


<img width="572" height="701" alt="image" src="https://github.com/user-attachments/assets/ad4e86a2-d4ea-485b-a26f-195038ece207" />


## Part II: Supervised Classification


## Part III: 



