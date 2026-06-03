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

>[!WARNING]
>This step takes approximately 60 minutes to finish processing and will create an image file >100GB in size.

<img width="572" height="701" alt="image" src="https://github.com/user-attachments/assets/ad4e86a2-d4ea-485b-a26f-195038ece207" />


## Part II: Supervised Classification
This tutorial will proceed through using the ArcGIS Pro classification wizard, using a supervised pixel-based random forest classfication. 


The Read Me file associated with this training repository includes additional details on classifying UAV imagery using either the ArcGIS Pro Deep Learning Package or R Studio. Both may achieve comparable or better results. 

>[CAUTION]
>The classificaiton and accuracy assessment performed during this training are for **training demsontrations only**, users should rely on a more statistically valid amount of training and validation (reference) data during standalone projects. This means, at a minimum there should be 50 samples per class for training and 50 smaples per class for validaiton. Point-based samples should not be used and are not allowed within the ArcGIS Pro classification wizard. 


### Step 1
Review the reference data shapefiles. Ensure that there are seperate training and validation files available for this section. The training and validation data have already been seperated in the provided package. If you do not have reference data yet, you can create these files using the **ArcGIS Pro Training Data Manager**.

**Each file must have the classname and classvalue fields**.

>[!TIP]
> For more information on classification in ArcGIS Pro visit:
> https://pro.arcgis.com/en/pro-app/3.4/help/analysis/image-analyst/the-image-classification-wizard.htm

### Step 2
Select the UAV composite created at the end of Part 1 in the Table of Contents.

Navigate to the Imagery Table and Open the **Classiication Wizard**. 


<img width="1045" height="251" alt="image" src="https://github.com/user-attachments/assets/f17dfad1-67e8-4603-908a-1b4711e7a7ec" />


### Step 3
Enter the classification configuration parameters for a supervised, pixel-based classifcation and click **Next >**. The Classification Schema can be imported from the training data shapefile (Default .gdb). 

<img width="585" height="1165" alt="image" src="https://github.com/user-attachments/assets/82555545-b70f-4be9-8dd2-b46c95773d22" />


### Step 4
Click next until you reach the **Train** step. 


Select the **Random Trees** classifier and edit the number of trees to **500**. When the processing is finished, expand the window at the bottom of the Image Classification Wizard Pane for Step #5. 


<img width="575" height="380" alt="image" src="https://github.com/user-attachments/assets/49dd908b-bfa4-4dc2-b880-2a0748ff2023" />

>[!CAUTION]
> The number of samples per class parameter in this step will randomly generate points within each provided training data polygon. If the number of samples per class exceeds the number of training data polygons, more than one point per polygon will be generated, thus biasing the classification (spatial autocorrelation). Typically, the number of randomly generated samples here should be equal to or less than the number of training data polygons available in the smallest class. 

### Step 5
Before clicking Next, expand the classifier metadata at the bottom and review the **train accuracy** and **variable importance** statistics. 

>[!CAUTION]
> This information can be used to understand the relative importance of the input variables as well as an initial understadning of the classification performance. This is the only place to retrieve this information, and it is lost after the classification wizard is completed and/or closed out. To save this information for future use, copy and paste it to an excel file. 


<img width="583" height="426" alt="image" src="https://github.com/user-attachments/assets/7555598a-5977-4abb-9d18-fa1d16c691ae" />


You can create a feature importance graphic in excel or another statistics software by cross-referencing this output with the band names and order (Image composite metadata / properties). 

### Step 6
Continue to the next step and Run the classifier. **Optionally** enter a name for an output classification definition file (.ecd). This is a unique file for ArcGIS Pro that can be used to classify (predict) other imagery of the same type through the standalone **Classify** tool. 


### Step 7
Review the classified map for qualtative (visual) approval. 

[picture]

### Step 8
continue to the accuracy assessment step


Set the number of validation ('reference') points to **150** and the Sampling Strategy to **Equalized Stratified Random**. 


This will ensure that the total number of validation points are equally distributed among the land cover classes (3 here, with 50 each now). 

[picture]

### Step 9
After you hit **Run** the error matrix will be added as a standalone table to the bottom of the Table of Contents as well as to the default geodatabase. 


Review this new table, including special attention to the class-specific and overall accuracies. 

[picture]

>[!Tip]
>You can rerun just the accuracy assessment step by modifying the output table name and clicking Run again. Because the validation points are randomized, this may result in (slightly) different accuracies for each class. Running the accuracy assessment at least 2-3 times can be a good way to more fully understand the error / confusion in the classification as well as ensure that the number or placement of the validation points is not biasing the accuracy assessment. **If you notice substantial changes in the accuracy for any specific class, you should further review the classified map and validation samples (iterate the process). 

### Step 10 (optional)
Copy and paste the full table to excel, then reformat the matrix into a more user-freindly visual. 


[picture]

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
