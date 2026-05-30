# Quantifying Landscape Structure – Fragmentation and Connectivity within KSWS

**Objective:** To quantify the degree of fragmentation within Keo Seima Wildlife Sanctuary (KSWS) in the years 2017 and 2024 (based on the outputs of tutorial 1b). The focus will be on the forest, open forest, grassland, and cropland cover types. Additionally, we will explore the change in core forest area within KSWS between our focal years. This training will cover how to use previously generated landcover datasets to understand the landscape structure (composition and configuration) within each year, and how it has changed overtime, from 2017 to 2024. The training will be conducted in **ArcGIS Pro** and **Microsoft Excel**. 


**Required:** 
- Classified land cover raster layers for 2017 and 2024 OR ArcGIS Pro Project File (Training 2026 DATA folder).
- Provided Microsoft Excel spreadsheet 'Landcover2024_SumStats_Excel' (Training 2026 DATA folder).

### Optional Pre-Processing:
This section relies on a simplifed classification scheme in comparison to the raster layers used throughout the earlier tutorials. The original 10-class schema for Keo Seima must be **recoded** to: Cropland, Developed, Forest, Open Forest, Grassland, and Water. The provided ArcGIS Pro package contains both the classified rasters from the previous tutorials as well as the recoded (simplifed) classified rasters.
- The Cropland class contains the following classes: Cashew, Cassava, Rubber, Paddy Rice

#
# Fragmentation (Part I)

### Step 1
Convert the simplified (6 class) land cover **rasters to polygons** (vector), then **Dissolve** by gridcode. 
<img width="868" height="952" alt="image" src="https://github.com/user-attachments/assets/53c5e588-9d14-4b72-a7e1-d9ad2ebf8a7e" />

### Step 2
Run the **Multipart to Singlepart** tool on the vector layers from Step #1 to disconnect the multipart polygons for each land cover classes. 

<img width="1030" height="712" alt="image" src="https://github.com/user-attachments/assets/9f9965ae-e5e9-4e7c-aaa5-73924a23ca16" />

### Step 3
Open the attribute tables for this new layer and **Add two new fields:**
1. Field Name = **PatchArea_ha** and Data Type = **Dobule**
2. Field Name = **EdgeLength_m** and Data Type = **Double**

>[!NOTE]
> The Patch Area (PA) is the area of the shape or polygon.
> The Edge Length (EL) is the shape or polygon perimeter.

<img width="975" height="266" alt="image" src="https://github.com/user-attachments/assets/88950a5b-6440-4c66-aab3-cc2c387bf21e" />

### Step 4
Save your table edits by clicking the Save button in the top ribbon and close out of Fields View in the table. 
<img width="975" height="263" alt="image" src="https://github.com/user-attachments/assets/4ff99aca-1628-4e31-82fd-300b65f2f70c" />

### Step 5
Calculate the two new fields: Patch Area and Edge Length
- Right click on the “PatchArea_ha” Field and select the **Calculate Geometry** button.
- Follow the settings in the figure below to calculate both the Path Area (in hectares, ha) and Edge Length (in meters, m) at the same time and **click OK.**

<img width="478" height="710" alt="image" src="https://github.com/user-attachments/assets/c3b5f831-f73a-4325-bc5c-2e16c932a3ca" />

>[!NOTE]
>We are interested in quantifying how fragmented the landscape is in KSWS in 2024. There are many ways to quantify landscape structure, specifically the distribution of patches across the landscape. In this tutorial, we are going to use a relatively simple approach to understanding patch distribution. This approach starts with calculating the area and perimeter of each patch. 

<img width="414" height="473" alt="image" src="https://github.com/user-attachments/assets/dc1a23a0-4ddc-42eb-be35-e00e30e94979" />

### Step 6
Calculate the class specific summary statistics. Right-click on the PatchArea_ha field and select **Summarize**. Follow the figure below to create a new summary table named **Landcover2024_SumStats**.

>This step allows us to calculate the intermediate variables for each land cover class before calculating the final fragmentation metrics in the next few steps.

<img width="872" height="1533" alt="image" src="https://github.com/user-attachments/assets/74da66e2-caec-48cc-9acc-02631a384406" />

### Step 7
**Open** the "Landcover2024_SumStats" summary table at the bottom of the table of contents. Here we have the summary statistics you just calculated in the previous step. These statistics will be used to quantify how fragmented (or patchy) the landscape was in 2024. To do that, **add two new attributes (fields) to this table, both with type double, then click save**.

- **PatchDensity** (PD) = number of patches / total landscape area
- **EdgeDensity** (ED) = edge length sum or total perimeter / total landscape area

  >[!NOTE]
  >Both PD and ED are used as common measurements of fragmentation. Both metrics can be compared across years to evaluate any change in patch size, number, and shape. “MEAN_PatchArea_ha” (from this table) is a third fragmentation metric that is a measure of the mean patch size, for each class type.


### Step 8
Right click on the new PatchDensity column and select **Calculate Field**. Set the expression below and click ok.


> **PatchDensity = !COUNT_PatchArea_ha! / 317,380.84**


The denominator of the equation is the Total Landscape Area. Based on our landcover, the Total Landscape Area of KSWS = **317,380.84 ha**. 


<img width="420" height="621" alt="image" src="https://github.com/user-attachments/assets/df5eae13-cd4f-43ee-a1d3-36e998ca4533" />


### Step 9
Repeat Step #8 to calculate the new **EdgeDensity** field. Set the expression equal to:


> **EdgeDensity = !SUM_EdgeLength_m! / 317,380.84**


Once again using the total landscape area for KSWS. 


<img width="456" height="677" alt="image" src="https://github.com/user-attachments/assets/0bf62edf-0250-4359-882a-7a52e7922bd2" />

#
# Fragmentation Indices in Microsoft Excel (Part II)

### Step 1
Open the provided "Landcover2024_SumStats_Excel" microsoft excel file

**OR** 

Copy your Landcover2024_SumStats ArcGIS table to a new, blank, spreadsheet.

### Step 2
Calcualte thr first fragmentation index. 

#
# Core Area in ArcGIS Pro (Part III)




# Conclusions
We have now quantified landscape fragmentation of KSWS and calculated the core area of the forest patches. Both of these processes help understand the landscape structure in KSWS, specifically the configuration. Given that we already understand the composition of KSWS from our landcover classification and change detection, gaining an understanding of the distribution and configuration of habitat patches can aid in habitat management and deforestation mitigation.  
Specifically, we looked at how the Forest, Open Forest, Grassland, and Cropland cover types are configured mostly in the year of 2024, but also in 2017. Fragmentation can occur as a result of anthropogenic activities such as agriculture expansion. However, it is important to note that fragmentation can also occur naturally on the landscape. A summary of the findings from the products we created in the tutorial can be found in the published (in press, Summer 2026) article. 


## **End**

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
