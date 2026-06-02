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
> Both PD and ED are used as common measurements of fragmentation. Both metrics can be compared across years to evaluate any change in patch size, number, and shape. “MEAN_PatchArea_ha” (from this table) is a third fragmentation metric that is a measure of the mean patch size, for each class type.


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
Fragmentation is the process of continuous natural landscapes being broken up into smaller, isolated patches) can be quantified or measured in a number of ways. One way to quantify it is to use the Patch Density, Edge Density, and Mean Patch Area metrics. 

### Step 1
Open the provided "Landcover2024_SumStats_Excel" microsoft excel file and navigate to the 'Fragmentation_2024' spreadsheet. This spreadsheet contains a copy of the Landcover2024_SumStats attribute table from the end of Part I (above). 

### Step 2
The blank column below 'FI' is used to calculate the **Fragmentation Index** based on the multiplication of the PatchDensity (PD, (patch count / total area in hectares)) and EdgeDensity (ED, (edge length in meters / total area in hectares)) columns using the following equation:
> = PD * ED


Drag this cell downwards to fill the remaining cells (land cover classes) with a FI value. 

>[!NOTE]
> The Fragmentation Index results for each class will be comparable across classes, and comparable across years as we will keep the units consistent throughout the calculations.

**The larger the FI value, the more fragmented the class type is across the landscape.**



### Step 3
We can now calculate another fragmentation metric that takes into account the mean patch area (MPA) in hectares. MPA has been found to be one of the most meaningful and useful metrics to assess landcover fragmentation (Flowers et al. 2020, Ma et al. 2023).

**Calculate the FIS column values** for each class in the corresponding column using the equation below. Select the PatchDensity, EdgeDensity, and MEAN_PatchArea_ha values in the first row to calculate the first FIS value and then drag the cell downward to fill the remaining four cells with a FIS value. 

> = (PD * ED) + (1/MPA)
>
>In this equation:
> PD = Patch Density (patches per hectare)
> ED = Edge Density (meters per hectare)
> MPA = mean patch area (in hectares)


<img width="2401" height="342" alt="image" src="https://github.com/user-attachments/assets/cb064569-3801-409d-9f3c-a97f1da7579b" />

>[!TIP]
> **Interpretation:**
>
> Fragmentation increases as PD and ED increase.
>
> 	a.	High PD = more, smaller patches
>
> 	b.	High ED = more patch perimeter relative to area, smaller patches
>
> Fragmentation increases as 1/MPA increases.
>
> a.	High 1/MPA = smaller average patch size
>
> b.	This is why we use (1/MPA) in the equation, because MPA alone increases as fragmentation decreases. So,
>   	inverting MPA ensures that this metric follows the same directional trend as PD and ED (increasing with
>   	fragmentation).
>  
>  The larger the FIS value, the more fragmented the class type is across the landscape. 

### Step 4
We are going to look at one more metric, which is a straightforward metric of habitat fragmentation: Number of Patches (NUMP). 

>[!NOTE]
>NUMP has been found to be one of the most important metrics when quantifying fragmentation. An increase or decrease in NUMP indicates an increase or decrease in fragmentation, respectively (Flowers et al. 2020). 

We have the NUMP already calculated for each class in our 2024 summary statistics table. In Excel, this data is in both the “Frequency” and “COUNT-PatchArea” columns. 

**Click the “NUMP” sheet at the bottom of your Excel window.**

In the first (upper) table, you will see the number of patches for each class for our focal years. 
<img width="1072" height="464" alt="image" src="https://github.com/user-attachments/assets/09520aca-2f9d-47ec-9312-3527185765f2" />

### Step 5
Use a One-Way ANOVA to test if the number of patches across classes (Forest, Open Forest, Grassland, Cropland) changed significantly between 2017 and 2024. 

-	Each column is a year and each row is a landcover class. 
-	Enable the Data Analysis Toolpak in Excel if it is not already enabled:
  -	Go to File > Options > Add-ins,
  -	At the bottom, next to “Manage Excel Add-ins” Click Go,
  -	Check **“Analysis Toolpak”** and click **Ok**.

<img width="373" height="505" alt="image" src="https://github.com/user-attachments/assets/face195d-9c75-4745-a770-e5ce22528819" />

    
-	Run the ANOVA
  -	Go to the **Data tab**
  -	Click **Data Analysis** (far right)
  -	Choose **ANOVA: Single Factor**, click OK
  -	For Input Range:
    -	Select your full table of values
    -	Check Labels in First Row if you included year names in row 1
    -
  - Choose **Grouped by: Columns**
  - Output to a **New Worksheet Ply** or a specified output range
  - Click **OK**.


<img width="170" height="135" alt="image" src="https://github.com/user-attachments/assets/a314a6a0-ec93-45a7-8930-a9c0f406751d" />


<img width="542" height="357" alt="image" src="https://github.com/user-attachments/assets/f5b303ab-74ce-42ff-a200-420eb8edef59" />

> [!NOTE]
> **Interpret the Results:**
> -	This model asked: _Is the mean number of patches per landcover class different between 2017 and 2024?_
> -	Each column (year) is a group, and contains 5 observations (NUMP values for 5 landcover classes). We are comparing the means across these 2 groups (years).
> -	P-value < 0.05 = statistically significant difference between years. This would mean that the number of patches per class changed significantly between groups (years).
>
> -	_What did we find?_ A p-value of greater than 0.05 would suggest that on average, the number of patches across landcover classes did not significantly change over time.  


### Step 6
Let’s look at the fragmentation results across our three years using the NUMP values. **Click on the “Plots” sheet at the bottom of your Excel window.**


Here we see some plots that we have created for this training in advance. These plots were created by selecting the data and inserting a recommended chart in Excel within the Insert tab. 

- NUMP: Number of Patches Across Time and Cover Type

- Fragmentation Index (FI): Fragmentation (FI) Across Time and Cover Type
  
- Fragmentation Index with Size (FIS): Fragmentation (FIS) Across Time and Cover Type

-	_What are some of the trends we are seeing?_

**Optionally:**
Go back into the ArcGIS project file and visualize the different landcover layers by year to note any noticeable areas of change. Layers can also be symbolized corresponding to the FI or FIS findings if that is of interest (e.g., more fragmented cover types in red and less fragmented in green, using a red/yellow/green color scheme). Additionally, you could symbolize cover types by NUMP ranges. 


#
# Core Area in ArcGIS Pro (Part III)

### Step 1
Navigate back to the ArcGIS Project file that we were previously working within (Part 1). 


Locate the **Core Area (Part 3) layer group** in your contents pane. Turn off all other layers in your contents pane other than “KSWS Boundary”. Turn on the Core Area (Part 3) layer group. 

Turn on the **Forest2024_Patches feature class** in the Core Area group layer in your contents pane. Explore this layer in the map. 

- This vector layer contains the 10 largest forest patches derived from the 2024 landcover map. There are also vector layers for the OpenForest, Grassland, and Cropland classes we have been focusing on during this tutorial, each containing the largest 10 patches within that class.  


### Step 2
Create the Core Area (Negative Buffer) 

Open the **Buffer Tool** and input the following settings and click Run:

>[!NOTE]
>This tool might take a few minutes to run, depending on your computer.

<img width="581" height="541" alt="image" src="https://github.com/user-attachments/assets/8281a5f7-da9e-49de-8edb-722170e6c10d" />

We want to gather the core area of our forest patches to then compare to previous years. To do this, we are going to use the Buffer tool with a negative distance that allows us to remove the edge of our forest polygons. The buffer distance can be set to any negative value to achieve a core area polygon. In this case, we are going to use a buffer of **-100m.**

>[!IMPORTANT]
>The buffer zone of 100m is a conservative threshold determined by modeled influences of edge effects in tropical forests, specifically where forests have abrupt or drastic edges often created by agricultural practices, similar to those in Cambodia. A buffer zone of 100m delineates our core forest areas. This distance is chosen based on previous studies that found tree recruitment, tree mortality, and canopy dynamics are significantly altered within this forest edge/buffer zone of 100m (Harper et al. 2005, Laurance et al. 1998b). Note: buffer zones can always be adjusted depending on your question of interest or if you have a wildlife species of interest you are modeling core habitat for. 


### STEP 3
Explore the Buffer Tool output in the map. You can see that there are still 10 rows in the attribute table, however the core area of the five patches are not contiguous anymore. There are several smaller patches of forest that are considered the core habitat of the 2024 forest patches. We might predict that the connectivity of the core area/habitat patches is lower than the overall forest patches as the core area patches are much more disjunct.

> If you were interested in analyzing the new core area “patches” (disconnected forest core area patches), you can run the Multipart to Singlepart Tool to break up the patches.


### STEP 4
Add a field to your new “Forest2024_CoreArea” feature class. 

Name the field “**Core_ha**” and set the Data Type to “**Double**”.

Click **Save**. Close out of Fields View.

### STEP 5
Right-click the “Core_ha” field and click “**Calculate Geometry**” to measure the area of the new core area in hectares. Input the following settings and click ok:


<img width="519" height="524" alt="image" src="https://github.com/user-attachments/assets/1e80b332-f054-4e5d-b444-b6a367679a32" />


### STEP 6
Explore the other core area layers.

Open the attribute tables of these additional layers and look to see how the core area values differ from what you calculated for 2024. 

Turn on each layer and explore how the core area patches differ across years in shape, size, and distribution. 

_What do you see? How do the forest patches compare to each other across the study period?_

<img width="532" height="556" alt="image" src="https://github.com/user-attachments/assets/0149a4f5-38b3-4035-b3b7-6fb15049b9c7" />


### STEP 7
Summarize the Forest Core Area in Each Year

Open the attribute table of “Forest2024_CoreArea”. Right click the “Core_ha” column and click Summarize. 

Input the following settings into the Summary Statistics Tool. Click Ok.

<img width="428" height="361" alt="image" src="https://github.com/user-attachments/assets/36ff314d-5a2b-42a1-adce-6bfde4b94be5" />

### STEP 8
Repeat Step #7 for the 2017  forest core area layers (or other years). Make sure to edit the Output Table name to match the year you are summarizing. 

Open each summary statistics table and look at how the total core area sizes differ across years. You can further this visualization through joining these tables and making a bar plot comparing the areas across years. We will not cover that in this tutorial however table join and chart instructions can be found online. 

>[!NOTE]
>These steps can be repeated across landcover classes to understand the core area changes overtime in non-forest classes. Furthermore, it would be interesting to see if the core area of croplands is expanding overtime, as we might expect with greater agricultural expansion. 

# Conclusions
We have now quantified landscape fragmentation of KSWS and calculated the core area of the forest patches. Both of these processes help understand the landscape structure in KSWS, specifically the configuration. Given that we already understand the composition of KSWS from our landcover classification and change detection, gaining an understanding of the distribution and configuration of habitat patches can aid in habitat management and deforestation mitigation.  
Specifically, we looked at how the Forest, Open Forest, Grassland, and Cropland cover types are configured mostly in the year of 2024, but also in 2017. Fragmentation can occur as a result of anthropogenic activities such as agriculture expansion. However, it is important to note that fragmentation can also occur naturally on the landscape. A summary of the findings from the products we created in the tutorial can be found in the published (in press, Summer 2026) article. 


## **End**

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
