# Header

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
><img width="131" height="120" alt="image" src="https://github.com/user-attachments/assets/ce842ec4-4cf7-4aff-b4f7-d503eec64f44" />


### Step 4


## Part II: Supervised Classification


## Part III: 



