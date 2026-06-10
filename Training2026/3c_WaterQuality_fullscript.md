# Full Code   
Feel free to copy and paste the full, working code if you encounter any issues with the training.

```js
// Import the shapefile of Cambodia water bodies and set the lake geometry to the shapefile  

var lake = ee.FeatureCollection(WaterBodies) 

Map.centerObject(lake, 11); 

function maskS2(image) { 
  var scl = image.select('SCL'); 
  var mask = scl.neq(3)   // cloud shadow 
    .and(scl.neq(8))      // cloud medium probability 
    .and(scl.neq(9))      // cloud high probability 
    .and(scl.neq(10))     // cirrus 
    .and(scl.neq(11));    // snow 

return image 
    .updateMask(mask) 
    .divide(10000) //Scales the data down  
    .copyProperties(image, ['system:time_start']); 
} 

var collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lake)
  .filterDate('2019-04-13', '2019-11-15')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .map(maskS2);

// Create median composite, median allows us to analyze typcal spectral conditions over the seclected date range 
var median = collection.median();

var visualization = {
  bands: ['B4', 'B3', 'B2'],
  min: 0.02879999950528145,
  max: 0.263949990272522,
};


//Display the median composite on the map. 
Map.addLayer(median, visualization, 'Median Pixel Composite', false);

// =============================== 
// 4. pH ESTIMATION EQUATION 
// =============================== 
 
// Coefficients for the pH equation 
var a = 7.866422; 
var b = 15.804186; 
var c = 1.366983; 
var d = 22.499174; 
var e = 5.929357; 
var f = 2.661643; 
var g = 8.793555; 
 
// Estimate pH 
var pH = median.expression( 
  'a - (b*B2) - (c*B3) + (d*B4) + (e*NIR) - (f*SWIR1) - (g*SWIR2)', 
  { 
    'B2': median.select('B2'), 
    'B4': median.select('B4'), 
    'B3': median.select('B3'), 
    'a': a, 
    'b': b, 
    'c': c, 
    'd': d, 
    'e': e, 
    'f': f, 
    'g': g, 
    'SWIR1': median.select('B11'), 
    'SWIR2': median.select('B12'), 
    'NIR': median.select('B8'), 
  } 
).rename('pH'); 

var phVis = {
  min: 5,
  max: 9,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
};

var pH_clipped = pH.clip(lake);
Map.addLayer(pH_clipped, phVis, 'pH');

var legend = ui.Panel({ style: { position: 'bottom-left', padding: '8px 15px' } });

var legendTitle = ui.Label({ value: 'Lake pH', style: { fontWeight: 'bold', fontSize: '16px' } });
legend.add(legendTitle);

// Function to create legend row 
var makeRow = function(color, label) {
  var colorBox = ui.Label({ style: { backgroundColor: color, padding: '8px', margin: '0 0 4px 0' } });
  var description = ui.Label({ value: label, style: { margin: '0 0 4px 6px' } });
  return ui.Panel({ widgets: [colorBox, description], layout: ui.Panel.Layout.Flow('horizontal') });
};

// Assign colors to pH values 
legend.add(makeRow('red', 'pH 5'));
legend.add(makeRow('orange', 'pH 6'));
legend.add(makeRow('yellow', 'pH 7'));
legend.add(makeRow('green', 'pH 8'));
legend.add(makeRow('blue', 'pH 9'));

Map.add(legend);

var stats = pH_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean Lake pH:', stats);

//Define bands   
var RED = median.select('B4'); 
var BLUE = median.select('B2'); 

// Restrict the analysis to clean water pixels, mask analysis to NDWI to eliminate the influence of any land/shoreline pixels   
var waterOnly = collection.map(function(img) { 
  var ndwi = img.normalizedDifference(['B3', 'B8']); 
  return img.updateMask(ndwi.gt(0)); 
}); 

var medianDO = waterOnly.median(); 

// Calculate DO
var DO = median.expression(
  '3.449174 + (0.538437 * B2) + (317.589192 * B3) - (345.811647 * B4) - (152.504728 * B8) + (223.886541 * B11) - (35.820433 * B12)',
  {
    'B2': median.select('B2'),
    'B3': median.select('B3'),
    'B4': median.select('B4'),
    'B8': median.select('B8'),
    'B11': median.select('B11'),
    'B12': median.select('B12')
  }
).rename('DO');

// Clip to shapefile
var DO_clipped = DO.clip(lake);

// Add to map
Map.addLayer(DO_clipped, { min: 0, max: 14, palette: ['red', 'yellow', 'green', 'blue'] }, 'Estimated DO');

var doStats = DO_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean DO:', doStats);

var doLegend = ui.Panel({ 
  style: { 
    position: 'bottom-right', 
    padding: '8px 15px' 
  } 
}); 

// Legend Title 
var doTitle = ui.Label({ 
  value: 'Dissolved Oxygen (mg/L)', 
  style: { 
    fontWeight: 'bold', 
    fontSize: '16px' 
  } 
}); 
doLegend.add(doTitle); 

// Legend row function 
var makeRow = function(color, label) { 
  var colorBox = ui.Label({ 
    style: { 
      backgroundColor: color, 
      padding: '8px', 
      margin: '0 0 4px 0' 
    } 
  }); 
  var description = ui.Label({ 
    value: label, 
    style: {margin: '0 0 4px 6px'} 
  }); 
  return ui.Panel({ 
    widgets: [colorBox, description], 
    layout: ui.Panel.Layout.Flow('horizontal') 
  }); 
}; 

// Add DO categories 
doLegend.add(makeRow('red', '0–3 mg/L')); 
doLegend.add(makeRow('orange', '3–5 mg/L')); 
doLegend.add(makeRow('yellow', '5–7 mg/L')); 
doLegend.add(makeRow('green', '7–10 mg/L')); 
doLegend.add(makeRow('blue', '>10 mg/L')); 

// Add legend to map 
Map.add(doLegend); 

// Select bands 
var RED = median.select('B4'); 
var GREEN = median.select('B3'); 

// Calculate turbidity index 
var turbidity = median.expression( 
  '43.466850 - (60.177839 * B2) + (28.437788 * B3) + (22.675374 * B4) - (35.594486 * B8) + (91.815632 * B11) - (84.077381 * B12)', 
  { 
    'B2': median.select('B2'), 
    'B3': median.select('B3'), 
    'B4': median.select('B4'), 
    'B8': median.select('B8'), 
    'B11': median.select('B11'), 
    'B12': median.select('B12') 
  } 
).rename('Turbidity'); 

// Clip to shapefile 
var Turbidity_clipped = turbidity.clip(lake); 

// Display turbidity layer  
var turbidity_clean = Turbidity_clipped.max(0); 

Map.addLayer( 
  turbidity_clean, 
  { 
    min: 0, 
    max: 70, 
    palette: ['blue', 'cyan', 'green', 'yellow', 'orange', 'red'] 
  }, 
  'Turbidity (NTU)' 
); 

// Turbidity stats  
var turbidityStats = turbidity_clean.reduceRegion({ 
  reducer: ee.Reducer.mean(), 
  geometry: lake.geometry(), 
  scale: 10, 
  maxPixels: 1e13 
}); 
print('Mean Turbidity:', turbidityStats); 

//Turbidity Legend  
var turbLegend = ui.Panel({ 
  style: { 
    position: 'top-right', 
    padding: '8px' 
  } 
});  

turbLegend.add(ui.Label({ 
  value: 'Turbidity (NTU)', 
  style: {fontWeight: 'bold'} 
})); 

var turbPalette = ['blue', 'cyan', 'green', 'yellow', 'orange', 'red']; 

var turbLabels = [ 
  '0 - 10', 
  '10 - 20', 
  '20 - 35', 
  '35 - 50', 
  '50 - 60', 
  '> 60' 
]; 

for (var i = 0; i < turbPalette.length; i++) { 
  turbLegend.add( 
    ui.Panel([ 
      ui.Label('', { 
        backgroundColor: turbPalette[i], 
        padding: '8px', 
        margin: '0 4px 0 0' 
      }), 
      ui.Label(turbLabels[i]) 
    ], ui.Panel.Layout.Flow('horizontal')) 
  ); 
} 
Map.add(turbLegend); 

// =============================== 
// CHLOROPHYLL-A ESTIMATION: NDCI (Normalized Difference Chlorophyll Index) 
// =============================== 
 
// Select bands 
var RED = median.select('B4'); 
var REDEDGE = median.select('B5'); 
 
// Calculate NDCI 
var ndci = REDEDGE.subtract(RED) 
  .divide(REDEDGE.add(RED)) 
  .rename('NDCI'); 
  
// Add Chlorophyll Layer  
var ndciVis = { 
  min: -0.4, 
  max: 0.6, 
  palette: [ 
    '0000FF', // low chlorophyll 
    '00FFFF', 
    '00FF00', 
    'FFFF00', 
    'FF0000'  // high chlorophyll 
  ] 
}; 
// Clip to shapefile  
var ndci_clipped = ndci.clip(lake); 
Map.addLayer(ndci_clipped, ndciVis, 'NDCI - Chlorophyll-a'); 

// Add Chlorophyll legend  
// =============================== 
// NDCI LEGEND 
// =============================== 
 
// Create panel 
var legend = ui.Panel({ 
  style: { 
    position: 'top-left', 
    padding: '8px 15px' 
  } 
}); 
 
// Title 
var legendTitle = ui.Label({ 
  value: 'NDCI - Chlorophyll-a', 
  style: { 
    fontWeight: 'bold', 
    fontSize: '16px', 
    margin: '0 0 4px 0', 
    padding: '0' 
  } 
}); 
 
legend.add(legendTitle); 
 
// Color bar 
var makeColorBar = function(palette) { 
  return ui.Thumbnail({ 
    image: ee.Image.pixelLonLat().select(0), 
    params: { 
      bbox: [0, 0, 1, 0.1], 
      dimensions: '200x20', 
      format: 'png', 
      min: 0, 
      max: 1, 
      palette: palette 
    }, 
    style: {stretch: 'horizontal', margin: '0px 8px'} 
  }); 
}; 
 
// Add color bar 
legend.add(makeColorBar([ 
  '0000FF', 
  '00FFFF', 
  '00FF00', 
  'FFFF00', 
  'FF0000' 
])); 
 
// Labels 
var labels = ui.Panel({ 
  widgets: [ 
    ui.Label('Low'), 
    ui.Label('Moderate', { 
      textAlign: 'center', 
      stretch: 'horizontal' 
    }), 
    ui.Label('High') 
  ], 
  layout: ui.Panel.Layout.flow('horizontal') 
}); 
 
legend.add(labels); 
 
// Add legend to map 
Map.add(legend);  

// Select bands 
var BLUE = median.select('B2') 
var GREEN = median.select('B3') 
var RED = median.select('B4') 

// Calculate EC index 
var conductivity = median.expression( 
  '(BLUE * RED) / GREEN', 
  { 
    'BLUE': BLUE, 
    'GREEN': GREEN, 
    'RED': RED 
  } 
).rename('Electrical_Conductivity'); 

// Clip to shapefile 
var conductivity_clipped = conductivity.clip(lake); 

// Display conductivity Layer  
var ecVis = { 
  min: 0, 
  max: 1.5, 

  palette: [ 
    'blue', 
    'cyan', 
    'green', 
    'yellow', 
    'orange', 
    'red' 
  ] 
}; 

Map.addLayer( 
  conductivity_clipped, 
  ecVis, 
  'Electrical Conductivity' 
); 

var ecLegend = ui.Panel({ 
  style: { 
    position: 'bottom-center', 
    padding: '8px 15px' 
  } 
}); 
 
var ecTitle = ui.Label({ 
  value: 'Electrical Conductivity', 
  style: { 
    fontWeight: 'bold', 
    fontSize: '16px' 
  } 
}); 

ecLegend.add(ecTitle); 
ecLegend.add(makeRow('blue', 'Very Low')); 
ecLegend.add(makeRow('cyan', 'Low')); 
ecLegend.add(makeRow('green', 'Moderate')); 
ecLegend.add(makeRow('yellow', 'Elevated')); 
ecLegend.add(makeRow('orange', 'High')); 
ecLegend.add(makeRow('red', 'Very High')); 

Map.add(ecLegend); 

// Select bands 
var NIR = median.select('B8'); 
var RED = median.select('B4'); 
 
// Calculate NDRE 
var ndre = median.expression( 
  '(NIR - REDEDGE) / (NIR + REDEDGE)', 
  { 
    'NIR': median.select('B8'), 
    'REDEDGE': median.select('B5') 
  } 
).rename('NDRE') 
  .clip(lake); 

// Clip to shapefile 
var vegetation_clipped = ndre.clip(lake); 
 
// Display  
var vegVis = { 
  min: -1, 
  max: 1, 
  palette: [ 
   'cyan', 
    'yellow', 
    'green', 
    'darkgreen', 
    'brown', 
  ] 
}; 
 
Map.addLayer( 
  ndre, 
  vegVis, 
  'Aquatic Vegetation' 
);   

var vegLegend = ui.Panel({ 
  style: { 
    position: 'top-center', 
    padding: '8px 15px' 
  } 
}); 

var vegTitle = ui.Label({ 
  value: 'Aquatic Vegetation (NDRE)', 
  style: { 
    fontWeight: 'bold', 
    fontSize: '16px' 
  } 
}); 
 
vegLegend.add(vegTitle); 
vegLegend.add(makeRow('yellow', 'Sparse')); 
vegLegend.add(makeRow('green', 'Moderate')); 
vegLegend.add(makeRow('darkgreen', 'Dense')); 
vegLegend.add(makeRow('brown', 'Very Dense')); 

Map.add(vegLegend); 

//We need to normalize all the parameters since they have different units  
//Normalize DO using a higher DO is better "score" 
var DO_score = DO.subtract(2) 
                 .divide(8) 
                 .multiply(100) 
                 .clamp(0,100); 
                  
// Normalize pH with ideal pH of 7.5 
var pH_score = ee.Image(100) 
  .subtract( 
    pH.subtract(7.5) 
      .abs() 
      .multiply(40) 
  ) 
  .clamp(0,100); 
   
//Normalize turbidity with lower turbidity being better 
var turb_score = ee.Image(100) 
  .subtract( 
    turbidity.divide(70) 
             .multiply(100) 
  ) 
  .clamp(0,100); 
   
//Normalize conductivity  
var EC_score = ee.Image(100) 
  .subtract( 
    conductivity.divide(0.2) 
      .multiply(100) 
  ) 
  .clamp(0,100); 
   
// Normalize aquatic vegetation  
var veg_score = ee.Image(100) 
  .subtract( 
    ndre.subtract(0.3) 
      .abs() 
      .multiply(250) 
  ) 
  .clamp(0,100); 
  
// Normalize chlorophyll a where higher chlorophyll-a means poorer water quality 
var chl_score = ee.Image(100) 
  .subtract( 
    ndci.subtract(-0.93) 
      .divide(0.98 - (-0.93)) // max - min 
      .multiply(100) 
  ) 
  .clamp(0,100);   


// Add the scores into a water quality index (WQI) with weighted parameters, you can change the weight to emphasize certain parameters 
var WQI = DO_score.multiply(0.30) 
  .add(turb_score.multiply(0.20)) 
  .add(pH_score.multiply(0.30)) 
  .add(EC_score.multiply(0.5)) 
  .add(veg_score.multiply(0.5)) 
  .add(chl_score.multiply(0.10)) 
  .rename('WQI'); 
  
//Display WQI 
Map.addLayer(WQI.clip(lake), { 
  min: 0, 
  max: 100, 
  palette: [ 
    'red', 
    'orange', 
    'yellow', 
    'lightgreen', 
    'green' 
  ] 
}, 'Water Quality Index'); 

//Add legend 
var legend = ui.Panel({ 
  style: { 
    position: 'bottom-left',   
    padding: '8px 15px' 
  } 
}); 

var palette = [ 
  'red', 
  'orange', 
  'yellow', 
  'lightgreen', 
  'green' 
]; 
 
var labels = [ 
  '0-20 Very Poor', 
  '20-40 Poor', 
  '40-60 Fair', 
  '60-80 Good', 
  '80-100 Excellent' 
]; 
 
var legend = ui.Panel({ 
  style: { 
    position: 'bottom-right', 
    padding: '8px 15px' 
  } 
}); 

// Create title 
var legendTitle = ui.Label({ 
  value: 'Water Quality Index', 
  style: { 
    fontWeight: 'bold', 
    fontSize: '16px', 
    margin: '0 0 8px 0' 
  } 
}); 

// Add title  
legend.add(legendTitle); 

// Add the color boxes 
for (var i = 0; i < palette.length; i++) { 
  legend.add( 
    ui.Panel([ 
      ui.Label('', { 
        backgroundColor: palette[i], 
        padding: '8px', 
        margin: '0 4px 4px 0' 
      }), 
      ui.Label(labels[i]) 
    ], ui.Panel.Layout.Flow('horizontal')) 
  ); 
} 
Map.add(legend); 
```