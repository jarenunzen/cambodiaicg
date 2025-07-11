# Full Code   
Feel free to copy and paste the full, working code if you encounter any issues with the training.

```js

/* 

Remote Sensing, Landscape Ecology, and Conservation

Keo Seima

2016-2024

*/


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



// Area of interest (aoi)
var aoi = geometry;

//------------------------------------------

// Define start and end date for the image collection. 

var start = ee.Date('2023-11-01');
var end = ee.Date('2024-03-01'); 

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


// Guide: 
// https://gis.stackexchange.com/questions/315554/google-engine-make-a-greenest-pixel-composite-for-sentinel-2 

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
Map.addLayer(ndvi_95.clip(geometry), {min:500, max: 2500, bands: ['B4_p30', 'B3_p30', 'B2_p30']}, 'NDVI Percentile Sentinel', false) 

// *Anything higher than 50% generates clouds for the ndvi reducer 
// This line 
print("Filtered collection by NDVI", ndvi_95); 

// Note: This is not the NDVI that will be used in the later classification, due to the format of the numbers (float instead of double)


var dataset = ee.Image('CGIAR/SRTM90_V4'); 
var elevation = dataset.select('elevation'); 
var slope = ee.Terrain.slope(elevation); 

//Map.addLayer(slope, {min: 0, max: 60}, 'slope'); 

var slope = slope.expression( 
  'slope * 1', { 
      'slope': slope.select('slope'), 
  }).rename("slope"); 

//---------------------------------------------------

// Define ROI 
var roi= geometry; 

//Date 
var startDate = ee.Date('2024-01-01'); 
var endDate =  ee.Date('2024-12-31'); 

// Create image collection of S-2 imagery for the period 2024-2024 
var S2 = ee.ImageCollection('COPERNICUS/S2_HARMONIZED')  //filter start and end date 
 
   //filter start and end date 
   .filter(ee.Filter.date(startDate, endDate)) 
   .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than',100) 
   //filter according to drawn boundary 
   .filterBounds(roi) 


// Function to calculate and add an NDVI band 
var addNDVI = function(image) { 
 return image.addBands(image.normalizedDifference(['B8', 'B4'] )); //'B8', 'B4' 
};   

// Add NDVI band to image collection 
var S2 = S2.map(addNDVI).select(['nd']); 


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

print('s1 SAR Image Collection (Annual)',sentinel1_vh); 

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


var byMonthYearS1 = ee.ImageCollection.fromImages(sequence_s1); 

var multibands1 = byMonthYearS1.toBands().clip(roi); 

var SAR = multibands1.rename(bandsName).clip(roi);//.rename(monLists1).clip(roi);// 

Map.addLayer(SAR,  {min: -25, max: -10}, 'SAR VH Image',0); 

//-----------------------------------------------------

// Calculate the NDVI manually:   NDVI = (B8 - B4) / (B8 + B4) 
var NDVI = ndvi_95.expression( 
  '(NIR - RED) / (NIR + RED)', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'RED':ndvi_95.select('B4_p30'), 
  }).rename("NDVI"); 

//print(NDVI, 'NDVI')  // This line can be used to check that the NDVI values are of the correct format and range 
var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']}; 
Map.addLayer(NDVI.clip(geometry), ndviParams, 'NDVI', false);   //remove forward slash to view layer


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

// Calculate EVI:   EVI = 2.5∗ (NIR – RED)/ (NIR+6.0 ∗ RED – 7.5 ∗ BLUE+1.0) 

var getEVI = ndvi_95.expression( 
  '2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))', { 
      'NIR': ndvi_95.select('B8_p30'), 
      'RED': ndvi_95.select('B4_p30'), 
      'BLUE': ndvi_95.select('B2_p30') 
  }).rename("EVI"); 

//print(getEVI, "EVI for KS") 
var eviViz = {min: -1, max: 1, palette: ['white', 'black']}; 


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


// Add New Indicies to Mosaic  
print(ndvi_95, "Composite image without SI") 

var newBands = ee.Image([NDVI, NBR, ndwi, getEVI, GI, MSI, SAVI, slope, elevation]); 

ndvi_95 = ndvi_95.addBands(newBands); 


var ndvi_95 = ndvi_95.addBands(SAR); 

var ndvi_95 = ndvi_95.addBands(multiband1_ndvi); 

print(ndvi_95, 'Final image with all bands (SAR+)')

//--------------------------------------------------

// Guide: https://developers.google.com/earth-engine/guides/classification 

// Merge the training samples into a singular collection 

var training = cassava.merge(cashew).merge(rubber).merge(water).merge(developed).merge(paddyrice).merge(forest).merge(openforest).merge(grassland); 

print("Training Data", training);  //this number should match the number of training samples (view at very top), unless there are over 5000 samples.


var bands = ["B2_p30", "B3_p30", "B4_p30", 'B5_p30', 'B6_p30', 'B7_p30', 'B8_p30', 'B8A_p30', 'B11_p30', 'B12_p30', 'NDVI', "NBR", "EVI", "GI", "MSI", "SAVI", "slope", 'elevation', 'SAR-01','SAR-02','SAR-03','SAR-04','SAR-05','SAR-06', 'SAR-07','SAR-08','SAR-09','SAR-10','SAR-11','SAR-12', '2020-01','2020-02','2020-03','2020-04','2020-05','2020-06', 
'2020-07','2020-08','2020-09','2020-10','2020-11','2020-12'];   


var input = ndvi_95.select(bands); 



var trainImage = input.sampleRegions({
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

// Guide:    https://gis.stackexchange.com/questions/427565/variable-importance-for-random-forest-classification-in-gee 
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



// Export the classified map to Google Drive 
Export.image.toDrive({ 
  image: classified, // Name of the classified image 
  description: 'KeoSeima_RF_Classification_2024',  // File name, will be a GeoTiff format 
  scale: 10,  // Spatial resolution/pixel size 
  maxPixels: 1e13,  // Sets a limit on data size... (i.e., max number of pixels) 1e12 is max 
  crs: 'EPSG:32648', // WGS 84/UTM Zone 48N 
  region: KSWS  // Change this back to 'geometry' to make the image area. 'KS' is buffer zone  
});


```
