Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/

# Step 1   
Import Data for this training including:

- Keo Seima Updated Boundary
- Cashew reference data
- Cassava reference data
- Developed reference data
- Forest reference data
- Open Forest reference data
- Paddy Rice reference data
- Rubber reference data
- Water reference data
- Grasslands reference data

*This information is located within the 'LandCover_ReferenceData_2022to2025' folder of this repository.* 

For more information on **how to upload spatial data (assets) to GEE**, please visit: 
(https://developers.google.com/earth-engine/guides/manage_assets)




# Step 2
Refresh the assets tab and import each of the new layers (assets) into a blank script. Rename the layers to:

- KSWS
- cashew
- cassava
- developed
- forest
- othercrop
- paddyrice
- rubber
- water
- grassland

![](https://github.com/user-attachments/assets/8178a4c7-c6a3-4609-88da-927386c4f775
)




# Step 3
Enter a new header (using comment syntax) for the script and save it as ‘KSWS_SAR_Training_2024’.


![](https://github.com/user-attachments/assets/6722e3cc-9413-4541-8f32-85a02e9ef2e4)

# Optionally
Add each of the reference data assets to the new map and visualize them as individual layers. 
```js
// Import Reference Data Points
Map.addLayer(cashew, {color: 'white'}, 'Cashew Reference Data'); 
Map.addLayer(cassava, {color: 'white'}, 'Cassava Reference Data'); 
Map.addLayer(rubber, {color: 'red'}, 'Rubber Reference Data'); 
Map.addLayer(paddyrice, {color: 'red'}, 'Paddy Rice Reference Data'); 
Map.addLayer(water, {color: 'blue'}, 'Water Reference Data'); 
Map.addLayer(developed, {color: 'grey'}, 'Developed Reference Data'); 
Map.addLayer(forest, {color: 'green'}, 'Forest Reference Data'); 
Map.addLayer(openforest, {color: 'green'}, 'Open Forest Reference Data'); 
Map.addLayer(grassland, {color: 'green'}, 'Grassland Reference Data'); 
```


# Step 4
Call a Sentinel-2 image collection and add it to the map as a median pixel composite for 2024. 
> [!NOTE]
> Note that this imagery is selected between November 2023 and February 2024.

**Google Earth Engine Sentinel-2 L1C Imagery:** 
(https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_HARMONIZED)

>What happens when you call imagery from May OR June 2024?

> [!NOTE]
> Dry Season: NOV - MARCH
> Wet Season: JULY - OCT

```js
// Median Pixel Composite
// Import Protected Area Boundary 

// Area of interest (aoi)
var aoi = KSWS;

// Define start and end date for the image collection. 

var start = ee.Date('2024-11-01');
var end = ee.Date('2025-03-01'); 

// Setinel 2 L1C Imagery 
// In this case "geometry" is a rectangle drawn by the user that is slightly larger than the buffer area 

var collection = ee.ImageCollection('COPERNICUS/S2_HARMONIZED')  // searches all sentinel 2 imagery pixels... 
 // searches all sentinel 2 imagery pixels... 
  .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20)) // ...filters on the metadata for pixels less than 20% cloud 
  .filterDate(start ,end) //... chooses only pixels between the dates you define here 
  .filterBounds(geometry); // ... that are within your aoi 

// s2_filter: filters all available sentinel 2 (s2) images. This function limits the spatial extent to the boundary and filter date range by start and end vars defined above,  
// so that the program does not have to load images across the entire globe.  

var s2_filter = collection 
.filterBounds(geometry) 
.filterDate(start, end); 

// print number of images in collection, given spatial/temporal filters 
// already defined 

var count = s2_filter.size(); 
print(count, 'Number of images in the initial Collection') 
var median = collection.median(); 

var visualization = { 
  bands: ['B4', 'B3', 'B2'], 
  min: 1500, 
  max: 2750, 
}; 

// Image Collection for defined time period 
Map.addLayer(s2_filter, visualization, 'Filtered Image Collection', false); 



Map.centerObject(geometry);  
Map.setOptions('SATELLITE'); 
```

<img width="1243" height="661" alt="image" src="https://github.com/user-attachments/assets/b4f7acf3-78e3-44d9-bd7b-e957364d37e0" />


# Step 5
Create a secondary pixel-based composite based on the ‘Greenest Pixel Composite’ (NDVI) percentile function. Compare the results between the median pixel composite and the Greenest Pixel Composite.

>[!TIP]
> Guide:
> (https://gis.stackexchange.com/questions/315554/google-engine-make-a-greenest-pixel-composite-for-sentinel-2)

```js
// Compute ndvi using red band 4 and NIR band 8a; calculated as 
// : (b8-b4)/(b8+b4) 

var ndvi = s2_filter.map(function(img){ 
var red = ee.Image(img.select('B4')); 
var nir = ee.Image(img.select('B8')); 
var ndvi = (nir.subtract(red)).divide(nir.add(red)).rename('ndvi'); 
return img.addBands(ndvi); 
}); 

// print metadata and attributes 
print(ndvi); 

// limit values returned to ONE value in the 50th percentile; not using max to avoid outliers 
var ndvi_95 = ndvi.reduce(ee.Reducer.percentile([30])); 
Map.addLayer(ndvi_95.clip(KSWS), {min:500, max: 2500, bands: ['B4_p30', 'B3_p30', 'B2_p30']}, 'NDVI Percentile Sentinel', false) 

// *Anything higher than 50% generates clouds for the ndvi reducer 
// This line 
print("Filtered collection by NDVI", ndvi_95); 
```
>[!NOTE]
>This is not the NDVI that will be used in the later classification, due to the format of the numbers (float instead of double)

# Step 6
Add the SRTM Elevation and Slope datasets to the script.
>[!NOTE]
>Data Available at: (https://developers.google.com/earth-engine/datasets/catalog/CGIAR_SRTM90_V4) 

```js

var dataset = ee.Image('CGIAR/SRTM90_V4'); 
var elevation = dataset.select('elevation'); 
var slope = ee.Terrain.slope(elevation); 

//Map.addLayer(slope, {min: 0, max: 60}, 'slope'); 

var slope = slope.expression( 
  'slope * 1', { 
      'slope': slope.select('slope'), 
  }).rename("slope"); 

print(slope, 'SRTM') 
```

# Step 7
Paste in the following code to generate the monthly median NDVI composites for 2024. 

>[!TIP]
>**Citation:** Fatchurrachman; Rudiyanto; Soh, N.C.; Shah, R.M.; Goh, E.G.; Setiawan,
>B.I.;Minasny, B. High-Resolution Mapping of Paddy Rice Extent and Growth Stages across
>Peninsular Malaysia Using a Fusion of Sentinel-1 and 2 Time Series Data in Google
>Earth Engine. Remote Sens. 2022, 14, x. https://doi.org/10.3390/xxxxx 

```js
// Define ROI 
var roi= KSWS; 

//Date 
var startDate = ee.Date('2024-01-01'); 
var endDate =  ee.Date('2024-12-31'); 

// Create image collection of S-2 imagery for the period 2024-2024 
var S2 = ee.ImageCollection('COPERNICUS/S2') 
   //filter start and end date 
   .filter(ee.Filter.date(startDate, endDate)) 
   .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than',100) 
   //filter according to drawn boundary 
   .filterBounds(roi) 

print(S2.limit(10)) 
print(S2.aggregate_array('SPACECRAFT_NAME')) 

// Function to calculate and add an NDVI band 
var addNDVI = function(image) { 
 return image.addBands(image.normalizedDifference(['B8', 'B4'] )); //'B8', 'B4' 
};   

// Add NDVI band to image collection 
var S2 = S2.map(addNDVI).select(['nd']); 

print('S2',S2.limit(10)) ; 

var NDVI=S2.select('nd'); 

// For month 
var month = 1; 

// Calculating number of intervals 
var months = endDate.difference(startDate,'month').divide(month).toInt(); 

// Generating a sequence  
var sequence = ee.List.sequence(0, months);  

var sequence_s1 = sequence.map(function(num){ 
  num = ee.Number(num); 
  var Start_interval = startDate.advance(num.multiply(month), 'month'); 
  var End_interval = startDate.advance(num.add(1).multiply(month), 'month'); 
  var subset = NDVI.filterDate(Start_interval,End_interval); 
  return subset.max().set('system:time_start',Start_interval); 
}); 

var byMonthYear = ee.ImageCollection.fromImages(sequence_s1); 

var multibandNDVI = byMonthYear.toBands().clip(roi); 

var bandsName=['2020-01','2020-02','2020-03','2020-04','2020-05','2020-06', 
               '2020-07','2020-08','2020-09','2020-10','2020-11','2020-12']              

var multiband1_ndvi = multibandNDVI.rename(bandsName).clip(roi); //(monList)// 

print('multiband NDVI Image', multiband1_ndvi);
```

# Step 8
Next, in this same section, paste in the following code to generate the monthly median SAR composites for 2024.

>[!TIP]
>https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S1_GRD

```js
/* 
  SAR Imagery 
*/ 

var bandsName=['SAR-01','SAR-02','SAR-03','SAR-04','SAR-05','SAR-06', 
               'SAR-07','SAR-08','SAR-09','SAR-10','SAR-11','SAR-12'] 

//s1 
var sentinel1_vh = ee.ImageCollection('COPERNICUS/S1_GRD') 
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH')) 
  .select('VH') 
  .filter(ee.Filter.eq('instrumentMode', 'IW')) 
  .filter(ee.Filter.eq('resolution_meters', 10)) 
  .filter(ee.Filter.date(startDate, endDate)) 
  .filter(ee.Filter.bounds(roi)) 

print('s1',sentinel1_vh); 

// For month 
var month = 1; 

// Calculating number of intervals 
var months = endDate.difference(startDate,'month').divide(month).toInt(); 

// Generating a sequence  
var sequence = ee.List.sequence(0, months);  

var sequence_s1 = sequence.map(function(num){ 
    num = ee.Number(num); 
    var Start_interval = startDate.advance(num.multiply(month), 'month'); 
    var End_interval = startDate.advance(num.add(1).multiply(month), 'month'); 
    var subset = sentinel1_vh.filterDate(Start_interval,End_interval); 
    return subset.median().set('system:time_start',Start_interval); 
}); 

print('sequence_s1',sequence_s1) 

var byMonthYearS1 = ee.ImageCollection.fromImages(sequence_s1); 

var multibands1 = byMonthYearS1.toBands().clip(roi); 

var SAR = multibands1.rename(bandsName).clip(roi);//.rename(monLists1).clip(roi);// 

Map.addLayer(SAR,  {min: -25, max: -10}, 'VH',0); 
```

# Step 9
Create spectral indices using the original NDVI percentile composite image **(Sentinel-2)**
>[!TIP]
>Based on guides: https://www.geo.fu-berlin.de/en/v/geo-it/gee/2-monitoring-ndvi-nbr/2-2-calculating-indices/index.html

>[!WARNING]
>If the cloud mask, and not the greenest pixel mosaic is used, each of the 'image' names and 'band' names will need to be adjusted.
>For example, '_p30' is added here. 

```js
// Calculate the NDVI manually:   NDVI = (B8 - B4) / (B8 + B4) 
var NDVI = ndvi_95.expression( 
  '(NIR - RED) / (NIR + RED)', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'RED':ndvi_95.select('B4_p30'), 
  }).rename("NDVI"); 

//print(NDVI, 'NDVI')  // This line can be used to check that the NDVI values are of the correct format and range 
var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']}; 
Map.addLayer(NDVI.clip(KSWS), ndviParams, 'NDVI');   //remove forward slash to view layer
```

# Step 10
Create the six additional Spectral Indices based on the NDVI (Sentinel-2) composite imagery and add these new bands to the Sentinel-2 image composite. The new composite, which is printed to the Console, should accurately list each of the desired multispectral image bands within its contents.
>[!NOTE]
>Spectral Indices can each be added to the map using the Map.addLayer function as shown above.
>This has removed from the training to ease processing and rendering. 

```js
// Calculate the Normalized Burn Ratio   NDVI = (B11 - B8) / (B11 + B8) 
var NBR = ndvi_95.expression( 
  '(SWIR - NIR) / (SWIR + NIR)', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'SWIR':ndvi_95.select('B11_p30'), 
  }).rename("NBR"); 

// Calculate the NDWI manually:   NDWI = (B8 - B12) / (B8 + B12) 
var ndwi = ndvi_95.expression( 
  '(NIR - SWIR) / (NIR + SWIR)', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'SWIR': ndvi_95.select('B12_p30'), 
  }).rename("ndwi"); 

var ndwiViz = {min: -1, max: 1, palette: ['44c9f1', '1637f1']}; 
// Map.addLayer(ndwi, ndwiViz, 'NDWI');  remove forward slash to view layer 
// Calculate EVI:   EVI = 2.5∗ (NIR – RED)/ (NIR+6.0 ∗ RED – 7.5 ∗ BLUE+1.0) 

var getEVI = ndvi_95.expression( 
  '2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'RED': ndvi_95.select('B4_p30'), 
      'BLUE': ndvi_95.select('B2_p30') 
  }).rename("EVI"); 

//print(getEVI, "EVI for KS") 
var eviViz = {min: -1, max: 1, palette: ['white', 'black']}; 

// Map.addLayer(getEVI, eviViz, 'EVI');  remove forward slash to view layer 

// Calculate Greeness Index: 

// GI = Green / Red 

var GI = ndvi_95.expression( 
  'GREEN / RED', { 
      'GREEN':ndvi_95.select('B3_p30'), 
      'RED': ndvi_95.select('B4_p30') 
  }).rename("GI"); 

//print(GI, 'Greeness Index') 

// Calculate Moisture Stress Index 

// MSI = SWIR1 / NIR 

var MSI = ndvi_95.expression( 
  'SWIR / NIR', { 
      'SWIR': ndvi_95.select('B11_p30'), 
      'NIR': ndvi_95.select('B8_p30') 
  }).rename("MSI"); 

//print(MSI, "Moisture Stress Index") 

// Calculate Soil Adjusted Vegetation Index 

// SAVI = ((NIR - Red) / (NIR + Red )) * (1 + L)     L = 0.5 

var SAVI = ndvi_95.expression( 
  '((NIR - RED) / (NIR + RED)) * (1 + 0.5)', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'RED': ndvi_95.select('B4_p30'), 
  }).rename("SAVI"); 

//print(SAVI, "Soil Adjusted Vegetation Index") 

// Add New Indicies to Mosaic  
print(ndvi_95, "Without SI") 

var newBands = ee.Image([NDVI, NBR, ndwi, getEVI, GI, MSI, SAVI, slope, elevation]); 

ndvi_95 = ndvi_95.addBands(newBands); 

print(ndvi_95, 'All bands BLXY (SAR)') 
```
# (Optionally)
Export the new image composite to your Google Drive account
>[!WARNING]
>This image file will be very large (20GB+) and may require a premium Google account to access due to storage limits.
>This file is note required to complete the remainder of the training. **Only the land cover map will be**

```js
Export.image.toDrive({
image: ndvi_95.select("B2_p50", "B3_p50", "B4_p50", 'B5_p50', 'B6_p50', 'B7_p50',
'B8_p50', 'B8A_p50', 'B11_p50', 'B12_p50', 'NDVI', "ndwi", "EVI", "GI", "MSI", "SAVI"),
description: 'KeoSeima_L1C_16Bands_DRY_2021',
scale: 10,
maxPixels: 1e13,
crs: 'EPSG:32648',
region: geometry
});
```

# Step 11
Add the SAR and NDVI monthly composites to the multispectral (Dry season) imagery from Part 1. This can be added using the following code at the end of the spectral index section (Step #8 of Part 1). You may need to overwrite the current print command for the ndvi_95 variable in the original code.
```js
var ndvi_95 = ndvi_95.addBands(SAR); 

var ndvi_95 = ndvi_95.addBands(multiband1_ndvi); 

print(ndvi_95, 'All bands BLXY (SAR)')
```

# Step 12
Overwrite the ‘bands’ variable at the start of the classification section. You will now use a combination of SAR, NDVI, and multispectral bands to conduct the classification.
```js
var bands = ["B2_p30", "B3_p30", "B4_p30", 'B5_p30', 'B6_p30', 'B7_p30', 'B8_p30', 'B8A_p30', 'B11_p30', 'B12_p30', 'NDVI', "NBR", "EVI", "GI", "MSI", "SAVI", "slope", 'elevation', 'SAR-01','SAR-02','SAR-03','SAR-04','SAR-05','SAR-06', 'SAR-07','SAR-08','SAR-09','SAR-10','SAR-11','SAR-12', '2020-01','2020-02','2020-03','2020-04','2020-05','2020-06', 
'2020-07','2020-08','2020-09','2020-10','2020-11','2020-12'];   
```




# Step 13
Import the reference data samples and merge these samples into a singular variable.
>[!TIP]
>Guide to GEE Classification: https://developers.google.com/earth-engine/guides/classification

>[!WARNING]
>These shapefiles must being in the EPSG: 4326 coordinate system (WGS 84) to be
>positionally accurate in GEE
>h(ttps://developers.google.com/earthengine/guides/table_upload#:~:text=4326%20before%20ploading.-,Upload%20a%20Shapefile,on%20your%20local%20file%20system.) 

```js

// Merge the training samples into a singular collection 

var training = cassava.merge(cashew).merge(rubber).merge(water).merge(developed).merge(paddyrice).merge(forest).merge(openforest).merge(grassland); 

print("Training Data", training);  //this number should match the number of training samples (view at very top), unless there are over 5000 samples.;  //this number should match the number of training samples (view at very top), unless there are over 5000 samples.
```



# Step 14
Split the training and validation data and run the Random Forest classification algorithm. This specific instance uses 70% of the reference data for each class to train the algorithm and reserves 30% of the reference data to validate the classification accuracy. 

>[!WARNING]
>Think about this partition in terms of the total amount of reference data that is
>available for each class. Each class should have at least 100 samples for training and
>validation (100 each) while not being weighted too unevenly across all classes.

```js
var trainImage = ndvi_95.sampleRegions({ 
  collection: training, 
  properties: ['lclu'], 
  scale: 10  
}); 

// Split the reference data into training and validation 
// Training: 70%  
// Validation: 30% 

var trainingData = trainImage.randomColumn(); 
var trainSet = trainingData.filter(ee.Filter.lessThan("random", 0.70)); 
var validationSet = trainingData.filter(ee.Filter.greaterThanOrEquals("random", 0.70)); 



// Classification  Model (Run Algorithm) 
var classifier = ee.Classifier.smileRandomForest(500).train({  // 500 trees  
  features: trainSet, //training data (# %) 
  classProperty: 'lclu', // attribute used to distinguish the classes 
  inputProperties: bands   
  }); 

// Run the Classification on Image Collection 
var classified = input.classify(classifier);
```

# Step 15
Following the classification, use the following code to run a feature importance test. The resulting graphic will define which input parameters (bands or ‘features’) are most capable of correctly distinguishing the land cover classes. *Remember, this test is specific to the random forest algorithm.
>[!TIP]
>Guide:(https://gis.stackexchange.com/questions/427565/variable-importance-for-random-forest-classification-in-gee) 

```js
print(classifier.explain(), 'Random Forest Classifier Properties:') 

var importance = ee.Dictionary(classifier.explain().get('importance')) 
var sum = importance.values().reduce(ee.Reducer.sum()) 

var relativeImportance = importance.map(function(key, val) { 
   return (ee.Number(val).multiply(100)).divide(sum) 
  }) 

print(relativeImportance, 'Relative Importance Parameter') 

var importanceFc = ee.FeatureCollection([ 
  ee.Feature(null, relativeImportance) 
]) 

var chart2 = ui.Chart.feature.byProperty({ 
  features: importanceFc 
}).setOptions({ 
      title: 'RF Variable Importance', 
      vAxis: {title: 'Importance'}, 
      hAxis: {title: 'Bands'} 
  }) 

print(chart2, 'Relative Importance') 
```

# Step 16
Finally, run an accuracy assessment on the land cover map. Use this accuracy assessment to review both the confusion (error) matrix and overall accuracy. If necessary, rerun the classification several times and record the average overall accuracy.
```js
// Accuracy Assessment 
var confusionMatrix = ee.ConfusionMatrix(validationSet.classify(classifier) 
.errorMatrix({ 
  actual:  'lclu',                           // Use this as the real values (given by the user) 
  predicted: 'classification'                // This is generated by the classifiers prediction 
})); 

// Values within the matrix will be the individual pixels contained within the provided polygons 
print('Confusion Matrix:', confusionMatrix); 
print('Overall Accuracy:', confusionMatrix.accuracy()); 

var exportAccuracy = ee.Feature(null, {matrix: confusionMatrix.array()}) 
```

# Step 17
If you are satisfied with the overall (and class-specific) accuracy, use the following function to export the map to Google Drive.
```js
// Export the classified map to Google Drive 
Export.image.toDrive({ 
  image: classified, // Name of the classified image 
  description: 'KeoSeima_RF_Classification_2025',  // File name, will be a GeoTiff format 
  scale: 10,  // Spatial resolution/pixel size 
  maxPixels: 1e13,  // Sets a limit on data size... (i.e., max number of pixels) 1e12 is max 
  crs: 'EPSG:32648', // WGS 84/UTM Zone 48N 
  region: KSWS  // Change this back to 'geometry' to make the image area. 'KS' is buffer zone  
});
```

# Step 18
Review the Error Matrix and Overall Accuracy. Record your results in a separate file (excel suggested).

![](https://static.observableusercontent.com/files/ffe6cc2746cf0cf1f14b0f821f24fa08e516e2781e5a5ab0ec9459dc7693debe7605852a2e94798f4e12a432a4e2687e403989547104b776159ff3321843a09e)


# Step 19
Optimize the classification using the Feature Importance test, reference data samples, and classification parameters.

![](https://static.observableusercontent.com/files/35e827595a416095378947b528c09ed52979504c136396d49acb899c4f31249d22b9954cbaa98ec06d38eee210f3c407e423b96855981ab234b3fef1db67a53a)


# Step 20
To rerun the classificaiton on a new year:
1. Change the **'startdate'** and **'enddate'** variables in Steps #4 and #7
2. Change the name of the output task (**description**) and file in Step #17
>[!WARNING]
>The user must be careful when applying reference data across multiple years, the best practice is the review and edit the reference data for **either each year** or at least every few years. Applying the same reference data from 2022 to 2016 imagery will result in **significant error!**
