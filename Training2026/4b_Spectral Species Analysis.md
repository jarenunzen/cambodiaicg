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
Add the header to your script.


```
# WCS/GCBC Spectral Species Diversity Tutorial	
# June 2026 Training

## GitHub BiodivMapR Package tutorial ## 
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_01.html
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_02.html
# https://github.com/jbferet/biodivMapR/tree/master/R
```

### Step 2
Install the required packages, then load them to yo

```
## Install packages in order
install.packages('rstudioapi')
install.packages('preprocS2')
install.packages('remotes')
remotes::install_github("jbferet/biodivMapR")

install.packages('terra')
install.packages('sf')


install.packages('biodivMapR')
install.packages('spinR')

install.packages("sfheaders")
install.packages("maptiles")
```



### Step 3
Load required packages and prepare input and output directories.

```
## load the packages
library(sf)
library(terra)
library(rstudioapi)
library(biodivMapR)
```

> [!NOTE]
>  You may need to verify that each of the pacakges (above) have been installed and activated.
> <img width="1196" height="814" alt="image" src="https://github.com/user-attachments/assets/8f2f024b-4d15-43da-bd5c-9a004ce3eba8" />




```
# clean workspace
rm(list = ls(all=TRUE)); gc()


if (rstudioapi::isAvailable()) 
  setwd(dirname(rstudioapi::getSourceEditorContext()$path))


# 1.1- define input & output directories
input_dir_vect <- './01_DATA/amazon/vector_data'
input_dir_rast <- './01_DATA/amazon/raster_data/sentinel-2'
dir.create(path = input_dir_rast, showWarnings = F, recursive = T)
dir.create(path = input_dir_vect, showWarnings = F, recursive = T)
```

### Step 4
The spatial extent of the area of interest (aoi) needs to be defined as a vector 
file. 

Such aoi can be defined with `QGIS`, `Google Earth`, or your favorite tool. 


Here an aoi located in Brazilian Amazon forest is defined. 
The site is located next to **Guedes, Japurá - State of Amazonas, Brazil**. 

```{r, eval=FALSE}
# 1.2- define area of interest
bbox <- st_bbox(c(xmin = -122.4, ymin = 37.7, xmax = -122.3, ymax = 37.8), 
                crs = st_crs(4326))

aoi_path <- file.path(input_dir_vect, "aoi.gpkg")



# 1.3. Convert the bbox directly to a spatial polygon object
poly_obj <- st_as_sf(st_as_sfc(bbox))

# 1.4. Write it to your GeoPackage
write_sf(obj = poly_obj, dsn = aoi_path, overwrite = TRUE)

```


### Step 5
A Sentinel-2 acquisition with minimal cloud cover needs to be identified. 
The [Copernicus Browser](https://browser.dataspace.copernicus.eu) is a useful 
service to explore the availability of Sentinel-2 acquisitions. 
In addition to spatio-temporal filters, it includes filters such as the maximum 
cloud cover.


Images with a maximum cloud cover  < 5%, are identified for August 2024. 
An acquisition from 23 August 2024 shows good conditions of acquisition. 

`preprocS2` allows downloading from multiple providers. 
Microsoft Planetary computer is currently the default option. 

The function `get_s2_raster` downloads Sentinel-2 data corresponding to the
spatio-temporal query. 

- `aoi_path`: path for vector layer. Must include a **unique polygon** 
- `datetime`: date of acquisition, provided as a _Date_ object.
- `stac_info`: list including provider identifier (default = `mpc`)
- `output_dir`: output directory where to store data
- `site_name`: this will allow identifying rasters later
- `options`: options including maximum cloud cover and higher level processing


>[!WARNING]
>Installing the 'jbferet' and 'preproS2' repo here may take some time due to the number of dependencies.
>This process might also take several installs. **Follow the prompts in the R Studio Console**. 


```
### download Sentinel-2 acquisition from online STAC resource
# 1.5- download  S2 acquisition
datetime <- as.Date('2024-08-23')
stac_info <- list('provider' = 'mpc')
site_name = 'amazon'

########
#Install 'remotes' if you don't have it already
if (!requireNamespace("remotes", quietly = TRUE)) {
  install.packages("remotes")
}

#Install preprocS2 from the official GitHub repository
remotes::install_github("jbferet/preprocS2")

library(preprocS2)
#######


options <- set_options_preprocS2(fun = 'get_s2_raster')
options$overwrite <- FALSE



list_files <- get_s2_raster(aoi_path = aoi_path, 
                            datetime = datetime, 
                            stac_info = stac_info, 
                            output_dir = input_dir_rast, 
                            site_name = site_name, 
                            options = options)

rast_path <- list_files$Refl_L2A                # S2 L2A reflectance
mask_path <- list_files$vegetation_mask         # S2 binary mask identifying vegetation, discarding clouds & shadows
```

>[!NOTE]
>The information resulting from the spatiotemporal query are saved in the 
directory defined with the variable `input_dir_rast`. 
It follows this file/folder structure:
>```
>├── collections
>├── plot_001.rds
>├── raster_samples
>├── amazon_001_2024-08-23.tiff
>├── amazon_001_2024-08-23_BIN.tiff
>├── amazon_001_2024-08-23_BIN_v2.tiff
>└── amazon_001_2024-08-23_SCL.tiff
>├── s2_tiles_amazon.rds
>└── s2_footprint_amazon.gpkg
>```
>

### Step 6
`collections/plot_001.rds` corresponds to the item collection resulting from 
the spatiotemporal query. 


Use the R function `readRDS` to access the content of the file.

```
readRDS('./01_DATA/amazon/raster_data/sentinel-2/collections/plot_001.rds')

# ###Items
# - features (1 item(s)):
#   - S2A_MSIL2A_20240823T144731_R139_T19MHT_20240823T215907
# - assets: 
# AOT, B01, B02, B03, B04, B05, B06, B07, B08, B09, B11, B12, B8A, datastrip-metadata, granule-metadata, inspire-metadata, product-metadata, rendered_preview, safe-manifest, SCL, tilejson, visual, WVP
# - item's fields: 
# assets, bbox, collection, geometry, id, links, properties, stac_extensions, stac_version, type
```



- `raster_samples` includes:


  ─ `amazon_001_2024-08-23.tiff`: L2A (surface reflectance) Sentinel-2 data
  
  ─ `amazon_001_2024-08-23_BIN.tiff`: binary mask derived from
  `amazon_001_2024-08-23_SCL.tiff` and focusing on vegetation class.


  ─ `amazon_001_2024-08-23_BIN_v2.tiff`: binary mask produced from alternative
  radiometric criteria, including NDVI mask to remove non vegetated pixels,
  Blue mask to remove hazy/cloudy pixels, and NIR mask to remove shaded pixels.


### Step 7
You can adjust threshold for cloudMask (B02), shadeMask (B08), and NDVIMask.
See [here](https://jbferet.github.io/biodivMapR/articles/biodivMapR_02.html#spectral-transformation) for additional information on how these masks work.
           ─ `amazon_001_2024-08-23_SCL.tiff`: scene classification provided by sen2cor 
           atmospheric correction.
        
           - `s2_tiles_amazon.rds` contains the Sentinel-2 tile ID for the scene
           
           - `s2_footprint_amazon.gpkg` contains the footprint of the Sentinel-2 tile ID
           
           The animated gif shows the Sentinel-2 acquisition. 
           The pixels masked by the binary mask `amazon_001_2024-08-23_BIN_v2.tiff` are 
           greyed.
           The surface reflectance in sentinel-2 L2A products is rescaled from 0-1 to 
           0-10000 and the data are stored in INT16 to save space. This allows 50% space 
           saved compared to FLOAT32.
           
           The following animation displays the sentinel-2 acquisition along with the 
           vegetation mask and with the following color dynamic: 
             
           - Red band: Band 04 (0 - 1000).
           
           - Green band: Band 03 (0 - 1000).
           
           - Blue band: Band 02 (0 - 1000).



### Step 8


# END

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
