# WCS/GCBC Spectral Species Diversity Tutorial				6/1/26, Maeve Kelley

[picture]

**Objective:**

We would like to use drone imagery of the AK village(s) in KSWS to assess differences in spectral diversity within and between land use classes, correlation between spectral diversity and plant species diversity, as well as the association of spectral diversity with key wildlife data. In order to begin to answer some of these questions, we must conduct a spectral species diversity analysis. **This analysis can be completed using R coding software**, multispectral imagery of the study area, ground vegetation plot data.


This training is sourced from the tutorial present in the **biodivMapR R package Github page (BiodivMapR, Jean-Baptiste Féret and Florian de Boissieu)**. This Github provides a clear, in depth tutorial with example data, code, and explanation:
- https://github.com/jbferet/biodivMapR/blob/master/vignettes/biodivMapR.Rmd


This training will be an overview of spectral species diversity analyses, **based on the tutorial by Féret**. 
This type of analysis should be adjusted based on the users’ questions and goals. For the R code and further details, see the Github pasted above. 
Here, we will walk through the processes and broad methods that can be used in this type of analysis. 


You can also review the Downey et al. 2023 paper and code from the REAM ME file of this repo for more context on the use and objectives of this analysis. 


Overview
-	We will turn multispectral imagery into maps of vegetation diversity, using spectral species. 
-	Spectral species are pixels clustered by spectral reflectance signatures. 
  o	Different plant species have different spectral properties, which can be detected and analyzed using remote sensing and modeling. 
  o	The spectral properties can be used to understand vegetation diversity.


**Required Data:**
- start...
  

**Site:** The three communities of Andong Kralong, Sre Lvi, and Pu Kong, located in Mondulkiri province. WCS x GCBC Project Site
<img width="1280" height="612" alt="image002" src="https://github.com/user-attachments/assets/12769687-6584-4f68-a26e-160d546e8201" />

**For more information on this project, visit:** 

https://www.gcbc.org.uk/project/recognising-and-rewarding-the-contribution-of-indigenous-knowledge-for-the-sustainable-management-of-biodiversity/


## Part I: Start...
