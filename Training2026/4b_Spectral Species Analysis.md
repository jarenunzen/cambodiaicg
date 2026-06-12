# WCS/GCBC Spectral Species Diversity Tutorial				
**6/1/26, Maeve Kelley**

<img width="881" height="690" alt="image" src="https://github.com/user-attachments/assets/1149703c-d66f-4a70-9d5b-98c42a177544" />



**Objective:**
We would like to use drone imagery of the AK village(s) in KSWS to assess differences in spectral diversity within and between land use classes, correlation between spectral diversity and plant species diversity, as well as the association of spectral diversity with key wildlife data. In order to begin to answer some of these questions, we must conduct a spectral species diversity analysis. **This analysis can be completed using R coding software**, multispectral imagery of the study area, ground vegetation plot data.


This training is sourced from the tutorial present in the **biodivMapR R package Github page (BiodivMapR, Jean-Baptiste Féret and Florian de Boissieu)**. This Github provides a clear, in depth tutorial with example data, code, and explanation:
- https://github.com/jbferet/biodivMapR/blob/master/vignettes/biodivMapR.Rmd


This training will be an overview of spectral species diversity analyses, **based on the tutorial by Féret**. 
This type of analysis should be adjusted based on the users’ questions and goals. For the R code and further details, see the Github pasted above. 
Here, we will walk through the processes and broad methods that can be used in this type of analysis. 


You can also review the **Downey et al. 2023 paper and code from the READ ME file** of this repo for more context on the use and objectives of this analysis. 


**Required Data:**
- Multispectral UAV Orthomosaic (Grid 1 used here)
- Classified Raster layers generated from UAV Orthomosaic

**Overview**
-	We will turn multispectral imagery into maps of vegetation diversity, using spectral species. 
-	Spectral species are pixels clustered by spectral reflectance signatures. 
  o	Different plant species have different spectral properties, which can be detected and analyzed using remote sensing and modeling. 
  o	The spectral properties can be used to understand vegetation diversity.

#
# **Table of Contents**
## Phase 1 Gathering Remotely Sensed Imagery and Preparing it 
1.	Define area of interest (AOI)
2.	Download Sentinel-2 image
3.	Create a mask to clean the imagery (e.g., cloud cover, shadows)


>[!NOTE]
> This can be completed using multispectral drone imagery such as the image above, for simplicity this tutorial just uses Sentinel-2 imagery.



## Phase 2 Converting the Image into Spectral Features 
Converting the image into spectral features can be done in multiple ways. The tutorial provides three different options. Depending on your goals, you may prefer one over another. 


  a.	Spectral Transformation – Standardized PCA (SPCA)


  b.	Spectral Indices 

  
  c.	Biophysical Properties


We will focus on the SPCA and the Spectral Indices. Principal component analysis (PCA) is often used by researchers. It is a technique that reduces the dimensionality of remote sensing data (Tsai et al. 2005). One might use a PCA over spectral indices for feature generation because PCAs use all of the spectral bands simultaneously whereas spectral indices focus on single vegetation characteristics (e.g., NDVI). However, PCAs are not always the best choice, so be sure to review analysis steps accordingly. 


## Phase 3 Biodiversity Mapping
1.	K-means clustering – defines the spectral species 
    a.	These clusters become the spectral “species”. The number of clusters must be defined (k). It has been generally recommended to use k=20 when using Shannon diversity metrics (Downey et al. 2023).


**More information:** https://gjkoplik.github.io/spectral_clustering/#k-means_clustering 


2.	Diversity index maps (rasters)


    a.	Alpha diversity == how diverse is the vegetation within each local window?


  	b.	Beta diversity == how different are vegetation communities between locations?

  
4.	Diversity maps – show spatial patterns of vegetation heterogeneity


## Flowchart
<img width="901" height="525" alt="image" src="https://github.com/user-attachments/assets/d46bcb9c-7cb0-4e07-a216-43732c78fcaa" />



## Phase 4: Linking Spectral Diversity and Vegetation Plot Diversity
Assuming you have collected ground vegetation plot data and calculated vegetation diversity metrics, you can then link the spectral species to the vegetation species diversity. 


1.	Define spectral diversity metrics at the plot scale (same scale as the vegetation plot data).
2.	Compute spectral diversity metrics at the plot scale.
3.	Statistically compare the spectral and vegetation plot diversities. This can be done at the plot level using various models (e.g., generalized linear models). Community diversity relationships can be modeled (beta diversity). The goal may be to quantify how well the spectral diversity predicts or compares to the real vegetation diversity. 

  
#
**Site:** The three communities of Andong Kralong, Sre Lvi, and Pu Kong, located in Mondulkiri province. WCS x GCBC Project Site
<img width="1280" height="612" alt="image002" src="https://github.com/user-attachments/assets/12769687-6584-4f68-a26e-160d546e8201" />

**For more information on this project, visit:** 

https://www.gcbc.org.uk/project/recognising-and-rewarding-the-contribution-of-indigenous-knowledge-for-the-sustainable-management-of-biodiversity/


#
## References
Downey, Sean S., et al. "An intermediate level of disturbance with customary agricultural practices increases species diversity in Maya community forests in Belize." Communications Earth & Environment 4.1 (2023): 428.


<img width="957" height="811" alt="image" src="https://github.com/user-attachments/assets/0fd4bdea-e359-454f-ad08-5c355d564324" />


**Figure from Downey et al. 2023, Supplemental Info.**

<img width="1013" height="804" alt="image" src="https://github.com/user-attachments/assets/8d4ca316-ddf6-432c-b32f-376beeb9be89" />


**Figure from Downey et al. 2023, example of spectral species raster (composition).**

<img width="1013" height="1037" alt="image" src="https://github.com/user-attachments/assets/d90872fa-6984-41f1-85fd-755806d753b1" />


**Downey et al. 2023 of the spectral species map (raster).**

<img width="1013" height="1126" alt="image" src="https://github.com/user-attachments/assets/663b114a-1685-4906-8b9a-8484dadc8aaf" />


**Downey et al. 2023, showing spectral species raster and vegetation plots.**

Tsai, F., E‐K. Lin, and Kunihiko Yoshino. "Spectrally segmented principal component analysis of hyperspectral imagery for mapping invasive plant species." International Journal of Remote Sensing 28.5 (2007): 1023-1039.


https://github.com/jbferet/biodivMapR/blob/master/vignettes/biodivMapR.Rmd


https://gjkoplik.github.io/spectral_clustering/#k-means_clustering 


https://rstudio-education.github.io/hopr/starting.html 


#
## Part I: The biodivMap R package
### Step 1
Add the header to your script. Once you have entered the name, date, and a short description (objective / site), save the R script. 


```
# WCS/GCBC Spectral Species Diversity Tutorial	
# June 2026 Training

## GitHub BiodivMapR Package tutorial ## 
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_01.html
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_02.html
# https://github.com/jbferet/biodivMapR/tree/master/R
```

<img width="971" height="309" alt="image" src="https://github.com/user-attachments/assets/770c014c-fc26-4b8c-a935-d9cc3145e65e" />


### Optional
Add in a description of the analysis, including the required inputs, flowchart, and description methods

```

# Notes of Spectral Species Workflow (for before using the Downey code) 
 
# inputs I will need: 
#   1) orthomosaic geotiff multispec file
#   2) land use classification raster (categorical)
#         use the multispec orthomosaic, training polygon 
#         data (with known land use history), and run the classification. 

# Export this as a categorical GeoTIFF. 
#   The raster will be used for patch metrics, edge density, 
#   land use proportions, and the spectral species composition per class


# Spectral Species Anlaysis
#   1) PCA
#   2) k-means clustering
#   3) create spectral species raster
#   4) compute spectral species composition per hexagon
#   5) compute spectral species composition per land use class


# Disturbance metrics
#   create hexagon grid
#   compute edge density per hexagon
#   compute land use proportions per hexagon
#   build adjacency matrix (W)
```


### Step 2
Install the required packages, then load them to your newly saved script. 

> [!NOTE]
> **Instlling R packages only needs to be completed once.** If you have these packages installed already, either remove these lines or place them within a comment block. 

```
# biodivMapR package
## https://github.com/jbferet/biodivMapR

#
install.packages("remotes")
# turn on remotes package via the pane (lower-right) to determine it was installed
# OR 
# library(remotes);

remotes::install_github('cran/dissUtils')
# turn on the package manually


# this may take a little bit and ask for updates
remotes::install_github('jbferet/biodivMapR')
# NOTE: choose CRAN only packages to update

#biodivMapR should not be visible in the Packages pane
######

# Optional (if you do not have these packages installed)
install.packages(c('rstudioapi', 'remotes', 'terra', 'sf', 'sfheaders', 'maptiles'))

```

>[!NOTE]
> If you recieve an error for installing the dependency **RTools** because of versioning, you will need to follow the prompts to download the most recent version of RTools as an independent application (from the web). 
> <img width="2731" height="631" alt="image" src="https://github.com/user-attachments/assets/699fbcff-0dfa-4a0c-aaab-e1ee646477c0" />



### Step 3
Load required packages for use in this analysis

```
#####
## load the packages for this analysis
library(terra)
library(sf)
library(biodivMapR)
library(landscapemetrics)
library(dplyr)
library(vegan)
library(spdep)
library(devtools)
library(remotes)

```

```
#Install 'remotes' if you don't have it already
if (!requireNamespace("remotes", quietly = TRUE)) {
  install.packages("remotes")
}


#Install preprocS2 from the official GitHub repository
remotes::install_github("jbferet/preprocS2")

library(preprocS2)
```


> [!NOTE]
>  You may need to verify that each of the pacakges (above) have been installed and activated.
> <img width="1196" height="814" alt="image" src="https://github.com/user-attachments/assets/8f2f024b-4d15-43da-bd5c-9a004ce3eba8" />


<img width="1471" height="1013" alt="image" src="https://github.com/user-attachments/assets/2195f925-a036-4775-bd78-c228e6a6570c" />


### Step 4
Define the working directory. We will use this file location for all inputs and outputs.

```
# Define the working directory
getwd()
setwd("C:/Users/btc28/OneDrive - USNH/Desktop/GCBC Training")
```


### Step 5
Specify the file path and file name for the raster image (Sentinel-2 or UAV orthomosaic). 

```
# Define the path to your desktop folder
# For many users, the pathway can be specified as '~/Desktop/GCBC Training'
desktop_path <- "C:/Users/btc28/OneDrive - USNH/Desktop/GCBC Training"

# Specify your raster (image) file name (Change "file.tif" to your actual file name and extension)
file_name <- "KeoSeima_Sentinel2_2025.tif"

# Combine them into the full file path
full_path <- file.path(desktop_path, file_name)

# Open/Load the raster image
# The 'rast' function works for almost all formats (.tif, .png, .img, .nc, etc.)
my_raster <- rast(full_path)
```

> [!WARNING]
>  The **biodivMapR** package uses a very specific folder structure and naming convention for its processing pipeline. After this point, all files will be saved relative to this directory and file name.


### Step 6
Print the metadata for your raster image to the console and plot the bands for your image to the Plots window. 

```
# Verify and visualize
print(my_raster)  # Displays metadata (resolution, CRS, dimensions)
plot(my_raster)   # Plots the raster image
```


### Step 8
Plot the remote sensing imagery with band combinations. 

Two options for displaying the imagery are provided here. 


```
## OPTION 1
# 'r=4, g=3, b=2' maps band 4 to Red, band 3 to Green, and band 2 to Blue
plotRGB(my_raster, 
        r = 4, g = 3, b = 2, 
        stretch = "lin",  # Options: "lin" (linear) or "hist" (histogram) to boost contrast
        main = "True Color Composite (Bands 4, 3, 2)")


## OPTION 2
# Calculate the mean and standard deviation of your raster layers
# (Using 2 standard deviations is the industry standard)
img_mean <- global(my_raster, "mean", na.rm = TRUE)$mean
img_sd   <- global(my_raster, "sd", na.rm = TRUE)$sd

# 2. Define the upper and lower bounds (Mean +/- 2*SD)
lower_bound <- img_mean - (2 * img_sd)
upper_bound <- img_mean + (2 * img_sd)

# 3. 'Clamp' the raster data so outliers are forced into these boundaries
my_clamped <- clamp(my_raster, lower = lower_bound, upper = upper_bound)

# 4. Plot the clamped image with a standard linear stretch
plotRGB(my_clamped, r = 4, g = 3, b = 2, 
        stretch = "lin", 
        main = "2-Standard Deviation Stretch")

```

**Option 1 Visual**


<img width="1581" height="1054" alt="image" src="https://github.com/user-attachments/assets/1740b428-abea-4521-98ff-365272dcbaec" />


**Option 2 Visual**


<img width="1581" height="1054" alt="image" src="https://github.com/user-attachments/assets/06d21c3d-8bcb-4dbb-9a3e-c3eb3d483df3" />



### Step 9
Access the classified (land cover) raster for your study area. This will be required later in the script for looking at the species diversity for each land cover class. 


In this tutorial, we use a 9-class raster based on the results of **training 1a.** The other crop class is not present 

```
## Load the classified raster (2024) for Keo Seima, clipped to the study region.
##  This is a 9 class raster
print("Forcing explicit load of Land Cover raster values...")

# Initialize the raster pointer
ms_pointer <- rast("C:/Users/btc28/OneDrive - USNH/Desktop/GCBC Training/Classified_embeddings_2024_GCBC.tif")

# Force terra to read the values directly into memory blocks
if  (readStart(ms_pointer)) {
  ms <- ms_pointer * 1  # Multiplying by 1 forces R to compute and cache all pixel data
  readStop(ms_pointer)
} else {
  ms <- ms_pointer
}

# Add explicit names for biodivMapR to prevent errors
names(ms) <- "Land_Cover"
```


#
## Part II: Standardized PCA
**producing spectral features from reflectance data: principle**

A pre-processing step is usually required in order to produce relevant features 
from remote sensing data. 


When using multispectral/hyperspectral optical imagery, several alternatives can 
be considered to produce these features.

-   **Spectral transformation**. This includes principal component analysis, 
minimum noise fraction, or any multidimensional method applying linear or 
non-linear transformations using the spectral information

-   **Spectral indices**. They consist in applying simple transformations 
(difference, ratio, normalized difference...) on a limited number of spectral 
bands available from sensor acquisition. 

-   **Biophysical properties**. They involve physical information obtained from 
radiative transfer models, combined with an inversion algorithm dedicated to 
regression. 


They aim at linking reflectance measured over part or all of the spectral 
range covered by a sensor, to one or multiple vegetation characteristics, such 
as LAI, leaf pigment and water content, LMA.


This tutorial describes a selection of options available for users interested in 
producing features from Sentinel-2 data to prepare for `biodivMapR` analysis. 



## reflectance data and vegetation mask

The image is assumed to be downloaded from the procedure described [previously](https://jbferet.github.io/biodivMapR/articles/biodivMapR_01.html){target="_blank"}

The binary mask corresponding to the image to be processed is optional. 
However we strongly recommend using a mask when possible in order to discard 
pixels which may not be relevant for ecological analysis. 
This is the case for shaded, cloudy and non-vegetated pixels when focusing on 
vegetation biodiversity. 
Artificial surfaces and mineral surfaces should also be masked as they usually 
strongly contribute to spectral variability and do not inform about biological 
diversity.

__Insufficient masking of irrelevant pixels may lead to poor estimation of vegetation diversity__: 
interfaces between vegetation non-vegetated regions (soil, artificial surface, 
water) will show very high local spectral heterogeneity.  
This will most probably lead to high spectral diversity, unrelated to species 
diversity. 

## spectral transformation

Standardized Principal Component Analysis (SPCA) can be applied on Sentinel-2 
images with the code below.

First, the mask produced with `preprocS2` during the previous step is refined 
using a combination of radiometric filters.

-   **NDVI filtering**: allows filtering to eliminate non-vegetated pixels. 
Nothing fancy so you may need to deal with mixed pixels. 
`NDVI_Thresh` defines the minimum NDVI value to be kept. 
By default, the NDVI is computed based on the NIR spectral band closest to 835 nm, 
and the RED spectral band closest to 670 nm.

-   **NIR filtering**: allows filtering of shadows and pixels with very low signal. 
`NIR_Thresh` defines the minimum NIR value to be kept. 
By default, the NIR spectral band closest to 835 nm is used.

-   **BLUE filtering**: allows filtering of clouds, based on the hypothesis that 
atmospheric scattering will lead to higher reflectance in the blue domain. 
`Blue_Thresh` defines the maximum Blue reflectance to be kept. 
By default, the Blue spectral band closest to 480 nm is used.

Normalization of reflectance data with Continuum Removal can be performed as 
additional pre-processing step. 
This is done by setting `Continuum_Removal = T` as input for `perform_PCA`.

**Continuum Removal** is available when working with reflectance data only. 
It requires information on the sensor spectral bands, defined by `input_rast_wl`. 
This procedure eliminates multiplicative effects on reflectance. 
It proved to be relatively efficient in decreasing the effect of changes in 
illumination within individual tree crowns, when using high spatial resolution
imaging spectroscopy over tropical forests.

Continuum Removal is recommended when processing high spatial resolution 
(spatial resolution $\leq$ 10 m) multi and hyperspectral data.

#
### Step 1
Start the second section of this analysis. 

```
## Section 2 ---------------------------------------------
### SPECTRAL SPECIES ANALYSIS (biodivMapR Workflow)
```


### Step 2
Create the director for the spectral species analysis (a package specific directory and file structure).

```
# Create a dedicated output folder 
Output_Dir <- "C:/Users/btc28/OneDrive - USNH/Desktop/GCBC Training/SpectralSpecies"
dir.create(Output_Dir, showWarnings = FALSE, recursive = TRUE)

# Dynamically count the bands in YOUR specific raster file
my_raster_temp <- rast(full_path)
num_bands      <- nlyr(my_raster_temp)
rm(my_raster_temp) # Free up memory

print(paste("Detected", num_bands, "bands in your raster file. Generating matching spectrum..."))
```

### Step 3
Search your input image for specific bands. These will be used for the principal component analysis (PCA or SPCA). This specific function differs from the original package in that it will call the bands that your image specifically has, not the standard set from Sentinel-2. 

```
# Create a generic wavelength vector matching your band count exactly
dynamic_wavelengths <- seq(from = 400, to = 400 + (num_bands - 1) * 50, by = 50)
```


### Step 4
Run the principal component analysis (PCA or SPCA options).

```
# --- Run PCA ---
#   10-30 seconds of processing, a loading bar will be created in the console
print("Running Principal Component Analysis natively on GeoTIFF...")
PCA_Output <- perform_PCA(
  input_raster_path = full_path,           # Path to your "KeoSeima_Sentinel2_2025.tif"
  input_rast_wl     = dynamic_wavelengths, # Perfectly matches your raster's bands
  output_dir        = Output_Dir,
  input_mask_path   = NULL,                
  Continuum_Removal = FALSE,               
  TypePCA           = "SPCA",              # Standardized PCA
  filetype          = "GTiff"              
)

# NOTE: Output of PCA_Output variable


# Extract the generated PCA file paths list
PCA_Files <- PCA_Output$PCA_Files
print("PCA Step Successful!")
```


Once spectral transformation is performed, principal component (PC) selection 
needs to be done. 
`biodivMapR` currently does not provide automated procedure for PC selection. 
Hence, PC selection is users responsibility. 
**We recommend selection based on visual inspection in order to identify spatial 
features highlighting changes in vegetation types / species composition.**
This procedure requires expertise on the ecosystem of interest. 
PC selection should ideally be validated with ground information.

Data-driven methods adjusted based on data distribution for a specific 
image/site, such as PCA, usually show poor generalization ability, and cannot be 
applied on other images. 
As explained earlier, PC selection is a subjective step which prevents from 
fully automated workflows. Moreover, spectral transformation such as PCA and MNF 
are usually computationally demanding.


#
### Step 5
Run the k-means clustering algorithm.

> [!TIP]
> There is a clustering algorithm used here to group clusters of species based on the SPA. This is an unsupervised classification for the SPA bands so that we can look at the heterogenity of clusters within each land cover class or defined patch.
>
> For more information on k-means clustering, visit: https://developers.google.com/machine-learning/clustering/kmeans/overview

```
# --- Map Spectral Species (Direct Native Clustering) ---
print("Clustering PCs into Spectral Species categories...")


# This calculates the distance of each pixel to the 20 cluster centers
km_predict <- function(data, kmeans_obj) {
  centers <- kmeans_obj$centers
  # Multi-band Euclidean distance calculation
  distances <- apply(centers, 1, function(x) {
    rowSums(sweep(data, 2, x, "-")^2)
  })
  # Find the index of the minimum distance (closest cluster center)
  return(max.col(-distances))
}
```

### Step 6



#
# END

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
