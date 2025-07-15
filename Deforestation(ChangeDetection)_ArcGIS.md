Measuring change over time is a fundamental application of Land Cover and Land Use maps. In this tutorial, we will work with the land cover maps (primarily in raster format) to look at the relative changes in natural lands (forest, open forest, and grassland) and croplands (cashew and cassava) throughout Keo Seima.

In the final steps, we will also look through a selection of summary statistics that can be generated using the combination of the land cover maps and change layer. 


# Step 1
Begin by downloading and unpacking the ArcGIS Pro Project Package (.ppkx) for this tutorial titled **'KeoSeima_LandCover_ChangeDetection'**.

ALTERNATIVELY 

You can choose to download and add the **'Change Detection INput Data'** layers to a blank map in ArcGIS Pro. 

>[!NOTE]
>Both the raster and vector layers will be needed to complete the final summary statistics for this guide. All datasets are the finalized products created during the previous **Post Processing** training. 

<img width="1728" height="1907" alt="image" src="https://github.com/user-attachments/assets/ad4fcaf0-a3c2-4586-bc66-a21aac1ccc10" />


# Step 2
Symbolize the land cover raster layers. Use the **Unique Values** symbology option, symbolize the layer based on the **'Value'** attribute, and provide a legend (labels). 

>[!NOTE]
>It will likely be helpful to wite down the land cover class values (1-9) somewhere easily
>accessible throughout the remainder of this tutorial. These interger values (gridcode/value) will
>be used in many of the upcoming steps.

# Step 3
**Import the Symbology** to your other land cover maps. Once all 9 have been symbolized, spend a few minutes exploring and comparing the data layers to gain an appreciation for the degree of change and relative locations of change in land cover. 

<img width="1625" height="1559" alt="image" src="https://github.com/user-attachments/assets/b44c9b43-e480-4771-b637-68004a902c5f" />

# Step 4
Open the geoprocessing toolbox and use the **Compute Change Raster** tool to generate the change in land cover for each respctive annual change.
- [ ] 2016 to 2017
- [ ] 2017 to 2018
- [ ] 2018 to 2019
- [ ] 2020 to 2021
- [ ] 2021 to 2022
- [ ] 2022 to 2023

Compute the **'Categorial Change'** for each time period based on: 
- **Changes from:** Forest, Open Forest, Grasslands
- **To Classes:** Cashew, Cassava, Rubber, Paddy Rice

<img width="427" height="1267" alt="image" src="https://github.com/user-attachments/assets/3388b8be-7b1f-4d01-9d1d-5e0671d238d6" />

# Step 5
Open the attribute table for each of the new change detection layers. 

Examine the various rows (features) and assess the types of 'change' attributes that are recorded. 

Row 7 (Other), are all the pixels that were determined to have no change between the two years. This row must be deleted.

Select the “Other” row and press “delete” at the top of the attribute table.

Perform this for each of the 7 change detection raster.

<img width="1092" height="310" alt="image" src="https://github.com/user-attachments/assets/40eab661-f153-422d-8892-2b83ec6edb54" />

# Step 6
Use the **Raster to Polygon** tool to convrt each of the change rasters to a vector layer. 

Select the **Class_Name** attribute as the **Field**.

Uncheck **"Simplify Polygons"** but Check **'Create multipart features'**

<img width="759" height="726" alt="image" src="https://github.com/user-attachments/assets/3447a1ea-4a5c-4b0c-a602-492dd80c2c59" />

# Step 7
Add a new field of type **Long** to each new polygon layer called **'YEAR'**. Use this field to define the year of change (forest loss). 

For example, if forest, open forest, or grasslands were converted to cashew in the 2017 to 2018 change layer, the change would be in 2018. 

<img width="1040" height="794" alt="image" src="https://github.com/user-attachments/assets/ac23dd00-a425-4a4c-9e74-0f056766fc3b" />

# Step 8
Use the **Dissolve** or **Pairwise Dissolve** tool to represent land cover change as a singular type for each of the respective years. 

Use the 'YEAR' field from Step 7 as the **'Dissolve Fields'** and Enable the 'Create multipart feature' setting. 

> Make sure to repeat this process to dissolve each of the 6 change layers.

<img width="764" height="927" alt="image" src="https://github.com/user-attachments/assets/cc464ff7-47a4-464f-a531-273dca00d920" />

# Step 9
**Merge** each of the change detection into a single output file. Mkae sure to name this file in a logical manner. 

# Step 10
> (1) Think about where this newly combined change layer may be overestimating the amount of chagne to croplands throughout the study region.
>
> (2) Also consider where we may be able to control for overestimated (comission) in the change layer.


