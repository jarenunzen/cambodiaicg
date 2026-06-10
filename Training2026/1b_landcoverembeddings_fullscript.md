# Full Code   
Feel free to copy and paste the full, working code if you encounter any issues with the training.
Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/


```js
/* Satellite Embedding Analysis | Keo Seima Wildlife Sanctuary */

// 1.0 IMPORT STUDY AREA & EMBEDDINGS
var aoi = KSWS; 
var dataset = ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL');

var image1 = dataset
  .filterDate('2024-01-01', '2025-01-01')
  .filterBounds(aoi);

var median = image1.median();

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



// 3.0 SPLIT TRAINING AND VALIDATION (70% / 30%)
var trainImage = median.sampleRegions({
  collection: training,
  properties: ['lclu'],
  scale: 10
});

var trainingData = trainImage.randomColumn();
var trainSet = trainingData.filter(ee.Filter.lessThan("random", 0.70)); 
var validationSet = trainingData.filter(ee.Filter.greaterThanOrEquals("random", 0.70));


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

var confusionMatrix_rf = ee.ConfusionMatrix(validationSet.classify(classifier_rf).errorMatrix({
  actual: "lclu",
  predicted: 'classification'
}));
print('RF Confusion Matrix:', confusionMatrix_rf);
print('RF Overall Accuracy:', confusionMatrix_rf.accuracy());

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
Map.addLayer(classified_rf.clip(KSWS), {palette: landcoverPalette, min: 1, max: 10}, 'Classification: Random Forest', false);
Map.addLayer(classified_xgb.clip(KSWS), {palette: landcoverPalette, min: 1, max: 10}, 'Classification: XGBoost', true);
Map.centerObject(KSWS);


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
