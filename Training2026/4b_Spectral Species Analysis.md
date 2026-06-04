# WCS/GCBC Spectral Species Diversity Tutorial				6/1/26, Maeve Kelley

[picture]

**Objective:**

We would like to use drone imagery of the AK village(s) in KSWS to assess differences in spectral diversity within and between land use classes, correlation between spectral diversity and plant species diversity, as well as the association of spectral diversity with key wildlife data. In order to begin to answer some of these questions, we must conduct a spectral species diversity analysis. **This analysis can be completed using R coding software**, multispectral imagery of the study area, ground vegetation plot data.


This training is sourced from the tutorial present in the **biodivMapR R package Github page (BiodivMapR, Jean-Baptiste Féret and Florian de Boissieu)**. This Github provides a clear, in depth tutorial with example data, code, and explanation:
- https://github.com/jbferet/biodivMapR/blob/master/vignettes/biodivMapR.Rmd


This training will be an overview of spectral species diversity analyses, **based on the tutorial by Féret**. 
This type of analysis should be adjusted based on the users’ questions and goals. For the R code and further details, see the Github pasted above. 
Here, we will walk through the processes and broad methods that can be used in this type of analysis. 


You can also review the **Downey et al. 2023 paper and code from the REAM ME file** of this repo for more context on the use and objectives of this analysis. 


**Required Data:**
- start...

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
a.	Specify dates
3.	Create a mask to clean the imagery (e.g., cloud cover, shadows)


>[!NOTE]
> This can be completed using multispectral drone imagery, the tutorial just uses Sentinel-2 imagery.



## Phase 2 Converting the Image into Spectral Features 
Converting the image into spectral features can be done in multiple ways. The tutorial provides three different options. Depending on your goals, you may prefer one over another. 


  a.	Spectral Transformation – Standardized PCA (SPCA)


  b.	Spectral Indices 

  
  c.	Biophysical Properties


We will focus on the SPCA and the Spectral Indices. Principal component analysis (PCA) is often used by researchers. It is a technique that reduces the dimensionality of remote sensing data (Tsai et al. 2005). One might use a PCA over spectral indices for feature generation because PCAs use all of the spectral bands simultaneously whereas spectral indices focus on single vegetation characteristics (e.g., NDVI). However, PCAs are not always the best choice, so be sure to review analysis steps accordingly. 


## Phase 3 Biodiversity Mapping
1.	K-means clustering – defines the spectral species 
    a.	These clusters become the spectral “species”. The number of clusters must be defined (k). It has been generally recommended to use k=20 when using
  	Shannon diversity metrics (Downey et al. 2023).


**More information:** https://gjkoplik.github.io/spectral_clustering/#k-means_clustering 


3.	Diversity index maps (rasters)
    a.	Alpha diversity  how diverse is the vegetation within each local window?
  	b.	Beta diversity  how different are vegetation communities between locations?

  
5.	Diversity maps – show spatial patterns of vegetation heterogeneity


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


```R
# WCS/GCBC Spectral Species Diversity Tutorial	
# June 2026 Training

## GitHub BiodivMapR Package tutorial ## 
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_01.html
# https://jbferet.github.io/biodivMapR/articles/biodivMapR_02.html
# https://github.com/jbferet/biodivMapR/tree/master/R
```

### Step 2





# END

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
