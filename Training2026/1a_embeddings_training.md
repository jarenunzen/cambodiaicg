Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/


This training section will be completed entirely in **Google Earth Engine**.



# Step 1   
Import the data for this training into Google Earth Engine including:
For more information on **how to upload spatial data (assets) to GEE**, please visit: 
(https://developers.google.com/earth-engine/guides/manage_assets)

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


<img width="1376" height="879" alt="image" src="https://github.com/user-attachments/assets/28e2a61f-9d79-4185-a94b-30ca71c41595" />



# Step 2
Refresh the assets tab and import each of the new layers (assets) into a blank script. Rename the layers to:


**Note: these variable names are case sensitive**

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
Enter a new header (using comment syntax below) for the script and save it as: 'KeoSeima_LandCoverClassification_2024'.


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
Map.addLayer(othercrop, {color: 'black'}, 'Other Crop Reference Data'); 
```

# Step 4
Load the image collection and clip the imagery to the date range and area of interest.

```js
// 1.0 IMPORT STUDY AREA & EMBEDDINGS
var aoi = KSWS; 
var dataset = ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL');

var image1 = dataset
  .filterDate('2024-01-01', '2025-01-01')
  .filterBounds(aoi);

var median = image1.median();
```

# Step 5 (Optional)
Multidimensional vectors (embeddings) are difficult to visualize but we can attempt to visualize three bands of the embedding space as an RGB layer.

```js
// Visualize three axes of the embedding space as an RGB. 
var visParams = {min: -0.3, max: 0.3, bands: ['A01', 'A16', 'A09']}; 
var median = image1.median().clip(aoi); 
Map.addLayer(median, visParams, '2025 embeddings'); 
```
![embeddings.png](..%2FImages%2Fembeddings.png)


> [!TIP]
> To learn more about the AlphaEarth Foundations (AEF) satellite embeddings dataset visit: [https://developers.google.com/earth-engine/tutorials/community/satellite-embedding-01-introduction](https://medium.com/google-earth/ai-powered-pixels-introducing-googles-satellite-embedding-dataset-31744c1f4650)
> <img width="1232" height="541" alt="image" src="https://github.com/user-attachments/assets/59c1e8d2-ef3f-4b51-9e9d-0906e95bf79d" />


# Step 6
Merge reference data and select embedding bands.
**Note: The first 1,000 samples of the merged reference data sample set will be printed to the console. This print function can be modified to review the sample size for each class, in the event of errors in using the reference data**

```js
//-----------------------------------------------------------
// 2.0 BANDS, TRAINING MERGE, & SPECTRAL PATTERN ANALYSIS
//-----------------------------------------------------------
// 2.1 Merge all training feature collections
var training = cashew.merge(cassava).merge(rubber).merge(paddyrice)
                      .merge(water).merge(developed).merge(forest)
                      .merge(openforest).merge(grassland).merge(othercrop);

print("Training Data Sample:", training.limit(1000));

// Define the 64 embedding bands
var bands = [
  'A00','A01','A02','A03','A04','A05','A06','A07','A08','A09','A10',
  'A11','A12','A13','A14','A15','A16','A17','A18','A19','A20','A21',
  'A22','A23','A24','A25','A26','A27','A28','A29','A30','A31','A32',
  'A33','A34','A35','A36','A37','A38','A39','A40','A41','A42','A43',
  'A44','A45','A46','A47','A48','A49','A50','A51','A52','A53','A54',
  'A55','A56','A57','A58','A59','A60','A61','A62','A63'
];

var input = median.select(bands);
```
# Step 7 (Optional)
Run a spectral pattern analysis using each of the land cover classes and all 64 of the AEF embeddings layer.

> [!TIP]
> Spectral pattern analyses can help users decipher which bands chaaracterize the greatest and least amount of uniqueness/seperability between defined land cover categories
> <img width="703" height="861" alt="image" src="https://github.com/user-attachments/assets/fceea64e-2951-4446-a053-3ec23987ed30" />


```js
// 2.2 SPECTRAL PATTERN ANALYSIS FUNCTION (ROBUST)
var runSpectralPatternAnalysis = function(imageInput, trainingVectors, bandNames) {
  
  // Sample the image across training locations to get band values associated with 'lclu'
  var sampledFeatures = imageInput.sampleRegions({
    collection: trainingVectors,
    properties: ['lclu'],
    scale: 10,
    geometries: false
  });

  // Map class integer IDs to readable string names
  var classMetadataList = [
    {id: 0, name: 'Cashew'},
    {id: 1, name: 'Cassava'},
    {id: 2, name: 'Rubber'},
    {id: 3, name: 'Paddy Rice'},
    {id: 4, name: 'Water'},
    {id: 5, name: 'Developed'},
    {id: 6, name: 'Forest'},
    {id: 7, name: 'Open Forest'},
    {id: 8, name: 'Grassland'},
    {id: 9, name: 'Other Crop'}
  ];

  // Dynamically build a combined reducer using the exact band names to guarantee correct dictionary outputs
  var baseReducer = ee.Reducer.mean().setOutputs([bandNames[0]]);
  var combinedReducer = ee.List(bandNames).slice(1).iterate(function(band, currentReducer) {
    return ee.Reducer(currentReducer).combine({
      reducer2: ee.Reducer.mean().setOutputs([band]),
      sharedInputs: false
    });
  }, baseReducer);

  // Loop through each class, compute the mean for all bands, and format into a FeatureCollection
  var profileFeatures = classMetadataList.map(function(cls) {
    // Isolate data points belonging exclusively to this specific land cover class
    var classSubset = sampledFeatures.filter(ee.Filter.eq('lclu', cls.id));
    
    // Safety check: Check if the subset actually contains features to avoid empty collection crashes
    var count = classSubset.size();
    
    var meanStats = ee.Dictionary(ee.Algorithms.If(
      count.gt(0),
      classSubset.reduceColumns({
        reducer: combinedReducer,
        selectors: bandNames
      }),
      // Fallback empty dictionary matching the structure if zero points exist
      ee.Dictionary.fromLists(bandNames, ee.List.repeat(0, bandNames.length))
    ));
    
    // Return a clean feature containing the band mean values and the class name label
    return ee.Feature(null, meanStats).set('label', cls.name);
  });
  
  // Filter out any completely empty classes that would distort the chart
  var profileCollection = ee.FeatureCollection(profileFeatures);

  // Define dynamic UI rendering properties for the chart
  var chartOptions = {
    title: 'Satellite Embedding Profile Pattern Analysis by Land Cover Class',
    hAxis: {
      title: 'Embedding Dimensions (Bands)',
      titleTextStyle: {italic: false, bold: true}
    },
    vAxis: {
      title: 'Mean Embedding Response Value',
      titleTextStyle: {italic: false, bold: true}
    },
    lineWidth: 2,
    pointSize: 0, 
    curveType: 'function' 
  };

  // Generate the chart object using the valid FeatureCollection
  var patternChart = ui.Chart.feature.byProperty({
    features: profileCollection,
    xProperties: bandNames,
    seriesProperty: 'label'
  }).setOptions(chartOptions);

  // Print chart to the GEE console window
  print('Spectral Pattern Analysis Chart:', patternChart);
};



// Execute the corrected pattern analysis function
runSpectralPatternAnalysis(input, training, bands);
```


# Step 8
Split the training and validation data and run the Random Forest classification algorithm. 

> [!CAUTION]
> Users should understand their class-specific amounts of reference data before selecting the realtive split between training and validation data. Each class must maintain a statistically valid amount of reference samples for the classification and accuracy assessment to be reliable.


```js
// 3.0 SPLIT TRAINING AND VALIDATION (70% / 30%)
var trainImage = median.sampleRegions({
  collection: training,
  properties: ['lclu'],
  scale: 10
});

var trainingData = trainImage.randomColumn();
var trainSet = trainingData.filter(ee.Filter.lessThan("random", 0.70)); 
var validationSet = trainingData.filter(ee.Filter.greaterThanOrEquals("random", 0.70));

```

# Step 9 
Run the Random Forest Algorithm for both classification and for mapping classification confidence.

> [!TIP]
> Review either of the following links to learn moure about classification and classification confidence analyses.
> Google Developer Page
>
> 
> (https://developers.google.com/earth-engine/apidocs/ee-classifier-smilerandomforest)
>
>
> Confidence Mapping
>
> 
> (https://developers.google.com/earth-engine/apidocs/ee-classifier-setoutputmode)


```js


//-----------------------------------------------------------
// 4.0 RANDOM FOREST CLASSIFICATION & CONFIDENCE SCORING
//-----------------------------------------------------------
var classifier_rf = ee.Classifier.smileRandomForest(500).train({
  features: trainSet,
  classProperty: 'lclu',
  inputProperties: bands
});

var classified_rf = input.classify(classifier_rf);

var probabilityClassifier_rf = ee.Classifier.smileRandomForest(500)
  .setOutputMode('MULTIPROBABILITY') 
  .train({
    features: trainSet,
    classProperty: 'lclu',
    inputProperties: bands
  });

var probabilities_rf = input.classify(probabilityClassifier_rf);

var confidence_rf = probabilities_rf.arrayReduce(ee.Reducer.max(), [0])
                                     .arrayGet([0]); 

// RF METRICS & EXPORTS
var importance_rf = ee.Dictionary(classifier_rf.explain().get('importance'));
var sum_rf = importance_rf.values().reduce(ee.Reducer.sum());
var relativeImportance_rf = importance_rf.map(function(key, val) {
  return (ee.Number(val).multiply(100)).divide(sum_rf);
});

var importanceFc_rf = ee.FeatureCollection([ee.Feature(null, relativeImportance_rf)]);
var chart_rf = ui.Chart.feature.byProperty({ features: importanceFc_rf }).setOptions({
  title: 'Random Forest Variable Importance',
  vAxis: {title: 'Importance'},
  hAxis: {title: 'Bands'}
});
print(chart_rf, 'RF Relative Importance');

Export.image.toDrive({
  image: classified_rf,
  description: 'KeoSeima_RF_Classification_2024',
  scale: 10,
  maxPixels: 1e13,
  crs: 'EPSG:32648',
  region: KSWS
});

Export.image.toDrive({
  image: confidence_rf,
  description: 'KeoSeima_RF_Confidence_2024',
  scale: 10,
  maxPixels: 1e13,
  crs: 'EPSG:32648',
  region: KSWS
});
```


<img width="1272" height="594" alt="AdobeExpressPhotos_6cc6029567554f4095690fc578013789_CopyEdited" src="https://github.com/user-attachments/assets/e6931abd-fe20-4de4-bcc0-3fc898e595f0" />


>[!NOTE]
>Take a second to explore the **random forest feature importance graph**. Using the pop-out **upper left** you can interact with or export the graphic and/or data. While this tutorial does not rely on feature reduction techniques or feature tuning, this can be a helpful method for better understanding the data and imporving the classification accuracy. **Classification is an iterative process!**


# Step 10
Create a confusion matrix and assess accuracy of the model.


> [!IMPORTANT]
> The overall and class-specific accuracies should be studied carefully in respect to the projects objectives. The classified maps should only be further analyzed or used for decision making if all types of accuracy are sufficient.



```js
var confusionMatrix_rf = ee.ConfusionMatrix(validationSet.classify(classifier_rf).errorMatrix({
  actual: "lclu",
  predicted: 'classification'
}));
print('RF Confusion Matrix:', confusionMatrix_rf);
print('RF Overall Accuracy:', confusionMatrix_rf.accuracy());
```

> [!TIP]
> For information on the use and understanding of the error matrix review the following paper:
> Russell G. Congalton, A review of assessing the accuracy of classifications of remotely sensed data, Remote Sensing of Environment,
Volume 37, Issue 1, 1991, Pages 35-46, https://doi.org/10.1016/0034-4257(91)90048-B.


![matrix.png](..%2FImages%2Fmatrix.png)




# Step 11
Run the Extreme Gradient Boosting (XGBoost) Classification and compare the Accuracy
```js

//-----------------------------------------------------------
// 5.0 XGBOOST CLASSIFICATION
//-----------------------------------------------------------
var classifier_xgb = ee.Classifier.smileGradientTreeBoost(100).train({
  features: trainSet,
  classProperty: 'lclu',
  inputProperties: bands
});

var classified_xgb = input.classify(classifier_xgb);

var importance_xgb = ee.Dictionary(classifier_xgb.explain().get('importance'));
var sum_xgb = importance_xgb.values().reduce(ee.Reducer.sum());
var relativeImportance_xgb = importance_xgb.map(function(key, val) {
  return (ee.Number(val).multiply(100)).divide(sum_xgb);
});

var importanceFc_xgb = ee.FeatureCollection([ee.Feature(null, relativeImportance_xgb)]);
var chart_xgb = ui.Chart.feature.byProperty({ features: importanceFc_xgb }).setOptions({
  title: 'XGBoost Variable Importance',
  vAxis: {title: 'Importance'},
  hAxis: {title: 'Bands'}
});
print(chart_xgb, 'XGBoost Relative Importance');

var confusionMatrix_xgb = ee.ConfusionMatrix(validationSet.classify(classifier_xgb).errorMatrix({
  actual: "lclu",
  predicted: 'classification'
}));
print('XGBoost Confusion Matrix:', confusionMatrix_xgb);
print('XGBoost Overall Accuracy:', confusionMatrix_xgb.accuracy());
```

# Step 12 (Optional)
Print the structure of the first decision tree to the console


>[!NOTE]
>GEE does not have a simmple way to generate a decision tree from these classification functions. Users are encouraged to complete these steps through the Python api for the function support of decision tree plotting. 

```js

//===========================================================
// EXTRACTION FUNCTION FOR TREE STRUCTURES (CLEAN FIX)
//===========================================================
var printFirstDecisionTree = function(classifier, modelName) {
  // Call .explain() to pull underlying model metadata
  var explanation = classifier.explain();
  
  // Extract the list of tree strings
  var treeList = ee.List(explanation.get('trees'));
  
  // Isolate the very first tree (Index 0)
  var firstTreeString = ee.String(treeList.get(0));
  
  // FIX: Let standard JavaScript handle the text label combination
  print('=== FIRST TREE STRUCTURE: ' + modelName + ' ===', firstTreeString);
};

// Execute for Random Forest
printFirstDecisionTree(classifier_rf, 'Random Forest');
```

# Step 13
Add layers to map with comprehensive legend.
```js

//-----------------------------------------------------------
// 6.0 LEGEND AND MAP DISPLAY
//-----------------------------------------------------------
var landcoverPalette = [
  'DBB024', // cashew (0)
  'EFDB9A', // cassava (1)
  'C8C1C0', // rubber (2)
  '048A52', // paddy rice (3)
  '2AC8FA', // water (4)
  'DC5057', // developed (5)
  '056322', // forest (6)
  '77AF23', // open forest (7)
  'BEFF5C', // grassland (8)
  '6D7E25'  // other crop (9)
];

var landcoverNames = ['Cashew', 'Cassava', 'Rubber', 'Paddy Rice', 'Water', 'Developed', 'Forest', 'Open Forest', 'Grassland', 'Other Crop'];

var legend = ui.Panel({ style: { position: 'bottom-left', padding: '8px 15px' } });
var legendTitle = ui.Label({ value: 'Legend', style: { fontWeight: 'bold', fontSize: '16px', margin: '0 0 4px 0' } });
legend.add(legendTitle);

var makeRow = function(color, name) {
  var colorBox = ui.Label({ style: { backgroundColor: '#' + color, padding: '8px', margin: '0 0 4px 0' } });
  var description = ui.Label({ value: name, style: { margin: '0 0 4px 6px' } });
  return ui.Panel({ widgets: [colorBox, description], layout: ui.Panel.Layout.Flow('horizontal') });
};

for (var i = 0; i < 10; i++) {
  legend.add(makeRow(landcoverPalette[i], landcoverNames[i]));
}
Map.add(legend);

Map.addLayer(KSWS, {color: 'white'}, 'Keo Seima Boundary');
Map.addLayer(classified_rf.clip(KSWS), {palette: landcoverPalette, min: 0, max: 9}, 'Classification: Random Forest', false);
Map.addLayer(classified_xgb.clip(KSWS), {palette: landcoverPalette, min: 0, max: 9}, 'Classification: XGBoost', true);
Map.centerObject(KSWS);
```
![lulc.png](..%2FImages%2Flulc.png)

# Step 14 (Optional data exploration)
Display the land cover classes based on classification confidence (reliability threshold).

>[!IMPORTANT]
>This optional funcitonality for viewing the newly classified layers based on classification confidence can be processing intensive and slow to load. **Users are recommended to turn off the visualization of these new layers or remove this section of the script to imporve the performance of the other sections.** These layers can be seperately downloaded and visualized in ArcGIS Pro or QGIS.



```js
//-----------------------------------------------------------
// 7.0 CONFIDENCE MASKING AND DISPLAY OPTIONS
//-----------------------------------------------------------
var threshold = 0.70; 
var confidenceMask = confidence_rf.gte(threshold);
var classified_rf_masked = classified_rf.updateMask(confidenceMask);
var classified_xgb_masked = classified_xgb.updateMask(confidenceMask);

Map.addLayer(confidence_rf.clip(KSWS), {
  min: 0.3, 
  max: 1.0, 
  palette: ['black', 'red', 'yellow', 'white']
}, 'RF Continuous Confidence Score', false);

Map.addLayer(confidenceMask.clip(KSWS), {
  min: 0, 
  max: 1, 
  palette: ['grey', 'green']
}, 'Confidence Mask (Green = Kept, Grey = Removed)', false);

Map.addLayer(classified_rf_masked.clip(KSWS), {
  palette: landcoverPalette, 
  min: 0, 
  max: 9
}, 'RF Classification (Masked < ' + (threshold * 100) + '%)', true);

//-----------------------------------------------------------
// 8.0 INTERACTIVE UI FOR PER-CLASS CONFIDENCE MASKING (COMPLETED)
//-----------------------------------------------------------

// IDs matching array index logic safely
var classMetadata = [
  {id: 0, name: 'Cashew', color: 'DBB024'},
  {id: 1, name: 'Cassava', color: 'EFDB9A'},
  {id: 2, name: 'Rubber', color: 'C8C1C0'},
  {id: 3, name: 'Paddy Rice', color: '048A52'},
  {id: 4, name: 'Water', color: '2AC8FA'},
  {id: 5, name: 'Developed', color: 'DC5057'},
  {id: 6, name: 'Forest', color: '056322'},
  {id: 7, name: 'Open Forest', color: '77AF23'},
  {id: 8, name: 'Grassland', color: 'BEFF5C'},
  {id: 9, name: 'Other Crop', color: '6D7E25'}
];

var classNamesList = classMetadata.map(function(item) { return item.name; });
var classMap = {};
classMetadata.forEach(function(item) { classMap[item.name] = item; });

var controlPanel = ui.Panel({
  layout: ui.Panel.Layout.flow('vertical'),
  style: {
    position: 'bottom-right',
    padding: '14px',
    width: '300px',
    backgroundColor: 'rgba(255, 255, 255, 0.95)'
  }
});

var panelTitle = ui.Label({
  value: 'Class-Specific Confidence Control',
  style: {fontWeight: 'bold', fontSize: '16px', margin: '0 0 4px 0'}
});
var panelDesc = ui.Label({
  value: 'Filter pixels based on the classification confidence of a specific land cover type.',
  style: {fontSize: '11px', color: 'gray', margin: '0 0 12px 0'}
});
controlPanel.add(panelTitle).add(panelDesc);

// Reactive callback function for UI controls
var updateMapLayers = function() {
  var selectedClassName = classSelector.getValue();
  var thresholdPct = confidenceSlider.getValue();
  if (!selectedClassName) return; // Guard clause if nothing is picked yet
  
  var decimalThreshold = thresholdPct / 100;
  var targetClassObj = classMap[selectedClassName];
  
  // Extract specific class probability layer from the multiprobability array output
  var arrayIndexPosition = targetClassObj.id; 
  var classSpecificConfidence = probabilities_rf.arrayGet([arrayIndexPosition]);
  
  var isTargetClass = classified_rf.eq(targetClassObj.id);
  var passesConfidence = classSpecificConfidence.gte(decimalThreshold);
  var finalClassMasked = classified_rf.updateMask(isTargetClass.and(passesConfidence));
  
  var layers = Map.layers();
  var probLayerName = selectedClassName + ' Raw Probability Map';
  var maskedLayerName = selectedClassName + ' Masked (Confidence ≥ ' + thresholdPct + '%)';
  
  // Track and dynamically refresh layers instead of stacking duplicates
  var probLayerExists = false;
  layers.forEach(function(layer) {
    if (layer.getName().indexOf('Raw Probability Map') !== -1) {
      layer.setEeObject(classSpecificConfidence.clip(KSWS));
      layer.setName(probLayerName);
      probLayerExists = true;
    }
  });
  if (!probLayerExists) {
    Map.addLayer(classSpecificConfidence.clip(KSWS), {min: 0, max: 1, palette: ['black', 'blue', 'yellow', 'white']}, probLayerName, false);
  }
  
  var maskedLayerExists = false;
  layers.forEach(function(layer) {
    if (layer.getName().indexOf('Masked (Confidence') !== -1) {
      layer.setEeObject(finalClassMasked.clip(KSWS));
      layer.setName(maskedLayerName);
      maskedLayerExists = true;
    }
  });
  if (!maskedLayerExists) {
    Map.addLayer(finalClassMasked.clip(KSWS), {palette: landcoverPalette, min: 0, max: 9}, maskedLayerName, true);
  }
};

// Construct UI components and wire up interactive triggers
var selectorLabel = ui.Label({value: '1. Select Target Class:', style: {fontWeight: 'bold', fontSize: '12px'}});
var classSelector = ui.Select({
  items: classNamesList,
  placeholder: 'Choose a class...',
  value: 'Forest',
  onChange: updateMapLayers
});

var sliderLabel = ui.Label({value: '2. Confidence Threshold (%):', style: {fontWeight: 'bold', fontSize: '12px', margin: '10px 0 0 0'}});
var confidenceSlider = ui.Slider({
  min: 0,
  max: 100,
  value: 70,
  step: 5,
  style: {stretch: 'horizontal'},
  onChange: updateMapLayers
});

// Assembly and rendering
controlPanel.add(selectorLabel).add(classSelector);
controlPanel.add(sliderLabel).add(confidenceSlider);
Map.add(controlPanel);

// Initialize the visualization on script execution
updateMapLayers();
```


# Step 15 
Use the 'Tasks' pane on the right side to export the classified map (raster layer) to Google Drive. Once the task has been started (Figure 2 below) users can close their GEE window (task will run on server side). 

>[!CAUTION]
>Classified raster layers of this size should be small enough (file size) to export and download for any Google Account (< 1GB each). For exporting multiple years of land cover maps OR **for exporting the embeddings mosaic** users will need a Google Account with greater stroage limited (> 100 GB). Such task may also take a longer time to complete (~50 minutes). 

**1.**
<img width="1181" height="479" alt="image" src="https://github.com/user-attachments/assets/a4631d0e-f201-4aee-bea9-f6d9d7a3ce4f" />


**2.**
<img width="1121" height="1448" alt="image" src="https://github.com/user-attachments/assets/96dcadba-b9ae-4e51-9558-0285943fe4e0" />


<img width="1142" height="109" alt="image" src="https://github.com/user-attachments/assets/35f6958d-0a03-42f6-b7c6-1efabaf12052" />


**3.**
<img width="1119" height="604" alt="image" src="https://github.com/user-attachments/assets/e62729a8-568d-43c5-b491-d5563b5d98d1" />


<img width="1170" height="538" alt="image" src="https://github.com/user-attachments/assets/28a15fb7-7d3c-4572-b1d1-50c987fdc7a2" />


<img width="2412" height="519" alt="image" src="https://github.com/user-attachments/assets/bbcfb898-cfb7-42d3-aafb-a67c3f7ba349" />

## **End**

>[!NOTE]
> This tutorial can be quickly revised to produce land cover maps for additional years (e.g., 2017). The deforestation training and subsequent sections relies on a secondary map (a sample map is provided). When adjusting the script, be sure to also review and/or update the reference data for each land cover class to match the change in date. Relying on a single set of reference data for clqssifying multiple years of imagery can lead to bias or error. 


#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
