# Full Code   
Feel free to copy and paste the full, working code if you encounter any issues with the training.

```js
// ===============================
// 1. DEFINE STUDY AREA
// ===============================
 
// Import the shapefile of Cambodia water bodies and set the lake geometry to the shapefile 
var lake = ee.FeatureCollection(WaterBodies)
 
Map.centerObject(lake, 11);
 
// ===============================
// 2. CLOUD MASK FUNCTION
// ===============================
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
  
// ===============================
// 3. LOAD SENTINEL-2 IMAGERY
// ===============================
 
var collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lake)
  .filterDate('2019-04-13', '2019-11-15')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .map(maskS2);
 
 
// Create median composite, median allows us to analyze typcal spectral conditions over the seclected date range
var median = collection.median();
 
// ===============================
// 4. CALCULATE WATER INDICES (NDWI and AWEInsh (Automated water extraction index) for pH analysis)
// ===============================
 
// NDWI
var ndwi = median.normalizedDifference(['B3', 'B8'])
  .rename('NDWI');
 
// AWEInsh
var aweinsh = median.expression(
  '4 * (GREEN - SWIR1) - (0.25 * NIR + 2.75 * SWIR2)',
  {
    'GREEN': median.select('B3'),
    'NIR': median.select('B8'),
    'SWIR1': median.select('B11'),
    'SWIR2': median.select('B12')
  }
).rename('AWEInsh');
 
// ===============================
// 5. pH ESTIMATION EQUATION
// ===============================
 
// Coefficients for the pH equation
var a = 1.36338;
var b = 0.00110;
var c = 0.00818;
var d = 0.00392;
var e = 0.00120;
 
// Estimate pH
var pH = median.expression(
  'a - b*AWEI + c*B2 - d*B4 + e*NDWI',
  {
    'AWEI': aweinsh,
    'B2': median.select('B2').multiply(10000),
    'B4': median.select('B4').multiply(10000),
    'NDWI': ndwi,
    'a': a,
    'b': b,
    'c': c,
    'd': d,
    'e': e
  }
).rename('pH');
 
var pH_adjusted = pH.add(3.7); //This adjuster is necessary since the original pH equation was created for Landsat imagery. We are using Sentinel-2 imagery for this anlaysis, but if we were to use Landsat, this adjuster would not be needed. 
 
// ===============================
// 6. DISPLAY pH MAP
// ===============================
 
var phVis = {
  min: 5,
  max: 9,
  palette: [
    'red',
    'orange',
    'yellow',
    'green',
    'blue'
  ]
};
 
 
// ===============================
// 7. Clip to shapefile 
// ===============================
 
var pH_clipped = pH_adjusted.clip(lake);
 
Map.addLayer(pH_clipped, phVis, 'pH');
 
// ===============================
// 8. CREATE LEGEND
// ===============================
 
var legend = ui.Panel({
  style: {
    position: 'bottom-left', //This can be edited to change the position of the legend
    padding: '8px 15px'
  }
});
 
var legendTitle = ui.Label({
  value: 'Lake pH',
  style: {
    fontWeight: 'bold',
    fontSize: '16px'
  }
});
 
legend.add(legendTitle);
 
// Function to create legend row
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
 
// Assign colors to pH values
legend.add(makeRow('red', 'pH 5'));
legend.add(makeRow('orange', 'pH 6'));
legend.add(makeRow('yellow', 'pH 7'));
legend.add(makeRow('green', 'pH 8'));
legend.add(makeRow('blue', 'pH 9'));
 
Map.add(legend);
 
// ===============================
// 9. PRINT pH STATISTICS
// ===============================
 
var stats = pH_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});
 
 
print('Mean Lake pH:', stats);
 
 
// ===============================
// DISSOLVED OXYGEN ESTIMATION
// ===============================
 
//Define bands  
var RED = median.select('B4');
var BLUE = median.select('B2');
 
//Restrict the analysis to clean water pixels, mask analysis to NDWI to eliminate the influence of any land/shoreline pixels  
var waterOnly = collection.map(function(img){
  var ndwi = img.normalizedDifference(['B3','B8']);
  return img.updateMask(ndwi.gt(0));
});
 
var median = waterOnly.median();
 
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
 
 
// Display
Map.addLayer(
  DO_clipped,
  {
    min: 0,
    max: 14,
    palette: ['red', 'yellow', 'green', 'blue']
  },
  'Estimated DO'
);
 
var doStats = DO_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry (),
  scale: 10,
  maxPixels: 1e13
});
 
print('Mean DO:', doStats);
 
// ===============================
// DO LEGEND
// ===============================
 
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
 
// ===============================
// TURBIDITY ESTIMATION
// ===============================
 
// Select bands
var RED = median.select('B4');
var GREEN = median.select('B3');
 
// Calculate turbidity index
var turbidity = median.expression(
  '(RED - GREEN) / (RED + GREEN)',
  {
    'RED': RED,
    'GREEN': GREEN
  }
).rename('Turbidity');
 
 
// Clip to shapefile
var Turbidity_clipped = turbidity.clip(lake);
 
// Display turbidity layer 
var turbidityVis = {
  min: -1,
  max: 1,
  palette: [
    'blue',
    'cyan',
    'yellow',
    'orange',
    'red'
  ]
};
 
Map.addLayer(
  Turbidity_clipped,
  turbidityVis,
  'Estimated Turbidity'
);
 
// Turbidity stats 
var turbidityStats = Turbidity_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});
 
print('Mean Turbidity:', turbidityStats);
 
//Turbidity Legend 
//Make sure to hit the gear on layers for DO and strech to 98%
var turbLegend = ui.Panel({
  style: {
    position: 'top-right',
    padding: '8px 15px'
  }
});
 
var turbTitle = ui.Label({
  value: 'Turbidity',
  style: {
    fontWeight: 'bold',
    fontSize: '16px'
  }
});
 
turbLegend.add(turbTitle);
 
turbLegend.add(makeRow('blue', 'Low'));
turbLegend.add(makeRow('cyan', 'Moderate'));
turbLegend.add(makeRow('yellow', 'Elevated'));
turbLegend.add(makeRow('orange', 'High'));
turbLegend.add(makeRow('red', 'Very High'));
 
Map.add(turbLegend);
 
// To display the layer better, go into layer settings (the gear icon next to the layer name), go to the range section, click the drop down labeled custom, and select stretch 98%. 
 
// ===============================
// CHLOROPHYLL-A ESTIMATION
// ===============================
 
// Select bands
var BLUE = median.select('B2');
var GREEN = median.select('B3');
var RED = median.select('B4');
 
// Calculate chlorophyll-a index
var chlorophyll = median.expression(
  '(BLUE - RED) / GREEN',
  {
    'BLUE': BLUE,
    'GREEN': GREEN,
    'RED': RED
  }
).rename('Chlorophyll_a');
 
// Clip to shapefile
var chlorophyll_clipped = chlorophyll.clip(lake);
 
// Add Chlorophyll Layer 
var chlVis = {
  min: -1,
  max: 1,
  palette: [
    'blue',
    'cyan',
    'green',
    'yellow',
    'red'
  ]
};
 
Map.addLayer(
  chlorophyll_clipped,
  chlVis,
  'Estimated Chlorophyll-a'
);
 
 
// Add stats 
var chlStats = chlorophyll_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake,
  scale: 10,
  maxPixels: 1e13
});
 
print('Mean Chlorophyll-a:', chlStats);
 
// Add Chlorophyll legend 
var chlLegend = ui.Panel({
  style: {
    position: 'top-left',
    padding: '8px 15px'
  }
});
 
var chlTitle = ui.Label({
  value: 'Chlorophyll-a',
  style: {
    fontWeight: 'bold',
    fontSize: '16px'
  }
});
 
chlLegend.add(chlTitle);
 
chlLegend.add(makeRow('blue', 'Low'));
chlLegend.add(makeRow('cyan', 'Moderate'));
chlLegend.add(makeRow('green', 'Elevated'));
chlLegend.add(makeRow('yellow', 'High'));
chlLegend.add(makeRow('red', 'Very High'));
 
Map.add(chlLegend); 
 
// ===============================
// ELECTRICAL CONDUCTIVITY
// ===============================
 
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
 
// Add conductivity stats 
var ecStats = conductivity_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});
 
print('Mean Electrical Conductivity:', ecStats);
 
// Add legend 
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
 
// ===============================
// AQUATIC VEGETATION (NDRE: Normalized Difference Red Edge), use NDRE over NDVI because it better visualizes chlorophyll content in plants 
// ===============================
 
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
 
 
//Add stats 
var vegStats = ndre.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});
 
print('Mean Aquatic Vegetation (NDRE):', vegStats);
 
//Legend
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
```