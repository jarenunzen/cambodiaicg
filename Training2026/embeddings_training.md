Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/


This training section will be completed entirely in Google Earth Engine.



# Step 1   
Import Data for this training including:

- Keo Seima Updated Boundary
1. Cashew reference data
2. Cassava reference data
3. Rubber reference data
4. Paddy Rice reference data
5. Water reference data
6. Developed reference data
7. Forest reference data
8. Open Forest reference data
9. Grasslands reference data
10. Other Crop reference data

*These data layers are located within the 'ReferenceData2025Embeddings' folder of this repository.* 

For more information on **how to upload spatial data (assets) to GEE**, please visit: 
(https://developers.google.com/earth-engine/guides/manage_assets)

<img width="1376" height="879" alt="image" src="https://github.com/user-attachments/assets/28e2a61f-9d79-4185-a94b-30ca71c41595" />




# Step 2
Refresh the assets tab and import each of the new layers (assets) into a blank script. Rename the layers to:
**these variable names are case sensitive**

- KSWS
- cashew
- cassava
- developed
- forest
- openforest
- othercrop
- paddyrice
- rubber
- water
- grassland

![imports1.png](..%2FImages%2Fimports1.png)


# Step 3
Enter a new header (using comment syntax) for the script and save it as: 'KeoSeima_LandCoverClassification_2024'


```js
'/* Satellite Embedding Analysis | Keo Seima Wildlife Sanctuary */'
```


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
Load image collection and clip imagery to date range and area of interest.
```js
//----------------------------------------------------------- 
// IMPORT STUDY AREA AND LOAD DATA
//----------------------------------------------------------- 

// Study area
var aoi = ksws 

// Load collection. 
var dataset = ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL'); 


// Get embedding images for two years. 
var image1 = dataset 
      .filterDate('2024-01-01', '2025-01-01') 
      .filterBounds(aoi); 
```

# Step 5 (Optional)
Multidimensional vectors (embeddings) are difficult to visualize but we can attempt to visualize
three bands of the embedding space as an RGB layer.
```js
// Visualize three axes of the embedding space as an RGB. 
var visParams = {min: -0.3, max: 0.3, bands: ['A01', 'A16', 'A09']}; 
var median = image1.median().clip(aoi); 
Map.addLayer(median, visParams, '2025 embeddings'); 
```
![embeddings.png](..%2FImages%2Fembeddings.png)
# Step 6
Merge reference data and select embedding bands.
```js
// Merge the training samples into a singular collection 
var training = forest.merge(openforest).merge(developed).merge(water).merge(cashew).merge(cassava).merge(rubber).merge(paddyrice).merge(grassland); 

// Embedding Bands
var bands = ['A00','A01','A02','A03','A04','A05','A06','A07','A08','A09','A10','A11','A12','A13','A14','A15','A16','A17','A18','A19', 
  'A20','A21','A22','A23','A24','A25','A26','A27','A28','A29','A30','A31','A32','A33','A34','A35','A36','A37','A38','A39','A40','A41','A42','A43','A44', 
  'A45','A46','A47','A48','A49','A50','A51','A52','A53','A54','A55','A56','A57','A58','A59','A60','A61','A62','A63'];   

var input = median.select(bands); 
```

# Step 7
Split the training and validation data and run the Random Forest classification algorithm. 
```js
//----------------------------------------------------------- 
// CREATE VALIDATION AND TRAINING DATA
//----------------------------------------------------------- 
var trainImage = median.sampleRegions({ 
  collection: training, 
  properties: ['lclu'], 
  scale: 10  
}); 

// Split the reference data into training and validation 
var trainingData = trainImage.randomColumn(); 
var trainSet = trainingData.filter(ee.Filter.lessThan("random", 0.50)); 
var validationSet = trainingData.filter(ee.Filter.greaterThanOrEquals("random", 0.50)); 
```

# Step 8 
Run the Random Forest Algorithm with 500 trees.
```js
//----------------------------------------------------------- 
// RUN THE CLASSIFICATION (RANDOM FOREST)
//----------------------------------------------------------- 

// Classification  Model (Run Algorithm) 
var classifier = ee.Classifier.smileRandomForest(500).train({  // 500 trees  
  features: trainSet, //training data (# %) 
  classProperty: 'lclu', // attribute used to distinguish the classes 
  inputProperties: bands   
  }); 

 
// Run the Classification on Image Collection 
var classified = input.classify(classifier).clip(aoi); 
```

# Step 9
Create a confusion matrix and assess accuracy of the model.
```js
//----------------------------------------------------------- 
// ACCURACY ASSESSSMENT
//----------------------------------------------------------- 
var confusionMatrix = ee.ConfusionMatrix(validationSet.classify(classifier).errorMatrix({ 
  actual: "lclu",                           // Use this as the real values (given by the user) 
  predicted: 'classification'               // This is generated by the classifiers prediction 
})); 


// Values within the matrix will be the individual pixels contained within the provided polygons 
print('Confusion Matrix:');
print(confusionMatrix.array()); 
print('Overall Accuracy:');
print(confusionMatrix.accuracy()); 

var exportAccuracy = ee.Feature(null, {matrix: confusionMatrix.array()}) 
```
![matrix.png](..%2FImages%2Fmatrix.png)
# Step 10
Add layers to map with comprehensive legend.
```js

//----------------------------------------------------------- 
// ADD LAYERS TO MAP
//-----------------------------------------------------------  
// ROI
Map.addLayer(ksws, {color: 'black'}, 'KSWS'); 


var landcoverPalette = [ 
  'DAE089', // Cashew 
  'BDA544', // Cassava 
  '827F72', // Rubber 
  'DE9228', // PaddyRice
  '287DDE', // Water
  'DE4328', // Developed
  '225E16', // Forest
  '28DE7D', // Openforest
  'FFA500' //Grassland
]

// Add a legend 
var legend = ui.Panel({  
  style: {  
    position: 'bottom-left',  
    padding: '8px 15px',  
    shown: true  
  }  
});  

// Create legend title 
var legendTitle = ui.Label({  
  value: 'Legend',  
  style: {  
    fontWeight: 'bold',  
    fontSize: '16px',  
    margin: '0 0 4px 0',  
    padding: '0'  
    }  
});  

// Add the title to the panel 
legend.add(legendTitle); 

// Creates and styles 1 row of the legend 
var makeRow = function(color, name) {  
  var colorBox = ui.Label({  
    style: {  
      backgroundColor: '#' + color,  
      padding: '8px',  
      margin: '0 0 4px 0'  
    }  
  });  

  // Create the label filled with the description text.  
  var description = ui.Label({  
    value: name,  
    style: {margin: '0 0 4px 6px'},  
  });  

  // return the panel  
  return ui.Panel({  
    widgets: [colorBox, description],  
    layout: ui.Panel.Layout.Flow('horizontal')  
  });  
};  

// Name of the legend 
var landcoverNames = ['Cashew', 'Cassava', 'Rubber', 'Paddy Rice', 'Water', 'Developed', 'Forest', 'OpenForest', 'OtherCrop', 'Grassland']; 

// Adding the specific number of rows to the legend 
for(var i = 0; i < 9; i++){ 
  legend.add(makeRow(landcoverPalette[i], landcoverNames[i])); 
} 

// Add legend to map 
Map.add(legend); 

// Add classified layer to the map 
Map.addLayer(classified, {palette: landcoverPalette, min:1, max:9}, 
'Classification: Random Forest', false); 
```
![lulc.png](..%2FImages%2Flulc.png)
# Step 11
Export classified image to Google Drive
```js
//----------------------------------------------------------- 
// EXPORT TO GOOGLE DRIVE
//----------------------------------------------------------- 
Export.image.toDrive({ 
  image: classified, // Name of the classified image 
  description: 'KSWS_Classification_2025_AEembeddings',  // File name, will be a GeoTiff format 
  scale: 10,  // Spatial resolution/pixel size 
  maxPixels: 1e13,  // Sets a limit on data size... (i.e., max number of pixels) 1e12 is max 
  crs: 'EPSG:32648', // WGS 84/UTM Zone 48N 
  region: ksws    
}); 
```
