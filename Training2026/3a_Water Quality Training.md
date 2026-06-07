## The following is based on a guide from NASA Applied Remote Sensing Training Program (ARSET) and data provided by WCS Cambodia. 
Several other trainings are also available for water quality monitoring using remote sensing through NASA ARSET. 

**Objective:**...

**Study Site:** Tonle Sap Lake, a shapefile for this boundary is provided via OpenDevelopment Cambodia
https://data.opendevelopmentmekong.net/dataset/water-bodies-in-cambodia

NASA AREST Training
- https://www.earthdata.nasa.gov/learn/trainings/monitoring-water-quality-inland-lakes-using-remote-sensing

# Step 0
Import water body shapefile.
![importwq.png](..%2FImages%2Fimportwq.png)

# Step 1
Define the study area for assessing water quality. Center the map to the study site.
```js
// Import the shapefile of Cambodia water bodies and set the lake geometry to the shapefile  

var lake = ee.FeatureCollection(WaterBodies) 

Map.centerObject(lake, 11); 
```

# Step 2
Define a Cloud Mask Function. Creates a reusable function that cleans each Sentinel-2 image before analysis. It uses the Scene Classification Layer (SCL) band to identify and remove pixels affected by cloud shadows, clouds, cirrus, and snow. 
```js
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
```

# Step 3
Load Sentinel-2 imagery. Define filter date to dry season to avoid heavy cloud coverage.
```js
var collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lake)
  .filterDate('2019-04-13', '2019-11-15')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .map(maskS2);

// Create median composite, median allows us to analyze typcal spectral conditions over the seclected date range 
var median = collection.median();
```

## Step 4
CALCULATE WATER INDICES (NDWI and AWEInsh (Automated water extraction index) for pH analysis).
```js
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
```
- **NDWI (Normalized Difference Water Index):** Uses green (B3) and near-infrared (B8) bands to identify open water. Values closer to 1 indicate water.
- **AWEInsh (Automated Water Extraction Index, no shadow):** A more complex index designed to separate water from other land cover types, particularly in areas where shadow can be confused with water.

# Step 5
 Estimate water body pH.
```js
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
    'a': a, 'b': b, 'c': c, 'd': d, 'e': e
  }
).rename('pH');

var pH_adjusted = pH.add(3.7);
```
> **Note:** The +3.7 adjustment is a sensor correction: the original equation was developed for Landsat, so a bias correction is needed when applying it to Sentinel-2 data. If using Landsat imagery, remove this adjustment.

# Step 6
Display the pH Map and clip to shapefile.
```js
var phVis = {
  min: 5,
  max: 9,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
};

var pH_clipped = pH_adjusted.clip(lake);
Map.addLayer(pH_clipped, phVis, 'pH');
```
![pH.png](..%2FImages%2FpH.png)

# Step 7 Add a pH Legend
 Add a pH Legend corresponding color to pH level.
```js
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
```

# Step 8
Calculate and print mean pH statistic.
```js
var stats = pH_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean Lake pH:', stats);
```

# Step 9
Estimate Dissolved Oxygen (DO).
```js
//Define bands   
var RED = median.select('B4'); 
var BLUE = median.select('B2'); 

// Restrict the analysis to clean water pixels, mask analysis to NDWI to eliminate the influence of any land/shoreline pixels   
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

// Add to map
Map.addLayer(DO_clipped, { min: 0, max: 14, palette: ['red', 'yellow', 'green', 'blue'] }, 'Estimated DO');

var doStats = DO_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean DO:', doStats);
```

# Step 10
Add a DO Legend.
```js
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
```
![do.png](..%2FImages%2Fdo.png)
# Step 11
Estimate Turbidity, estimated in NTU (Nephelometric Turbidity Unit. It is the standard unit of measurement used to quantify turbidity).
```js
// Calculate turbidity index 
var turbidity = median.expression(
  '(RED - GREEN) / (RED + GREEN)',
  {
    'RED': median.select('B4'),
    'GREEN': median.select('B3')
  }
).rename('Turbidity');

// Clip to shapefile 
var Turbidity_clipped = turbidity.clip(lake);

Map.addLayer(Turbidity_clipped, { min: -1, max: 1, palette: ['blue', 'cyan', 'yellow', 'orange', 'red'] }, 'Estimated Turbidity');

var turbidityStats = Turbidity_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean Turbidity:', turbidityStats);
```

# Step 12
Add a Turbidity Legend. Unlike the pH and DO legends which use numeric thresholds, this legend uses qualitative categories since the turbidity index is a relative (unitless) measure rather than a calibrated physical quantity.

```js
var turbLegend = ui.Panel({ style: { position: 'top-right', padding: '8px 15px' } });
var turbTitle = ui.Label({ value: 'Turbidity', style: { fontWeight: 'bold', fontSize: '16px' } });
turbLegend.add(turbTitle);

turbLegend.add(makeRow('blue', 'Low'));
turbLegend.add(makeRow('cyan', 'Moderate'));
turbLegend.add(makeRow('yellow', 'Elevated'));
turbLegend.add(makeRow('orange', 'High'));
turbLegend.add(makeRow('red', 'Very High'));

Map.add(turbLegend);
```
![turbidity.png](..%2FImages%2Fturbidity.png)
> **Tip:** To display the layer better, go into layer settings (the gear icon next to the layer name), go to the range section, click the drop down labeled custom, and select stretch 98%. 
# Step 13
Estimate Chlorophyll-a

```js
var chlorophyll = median.expression(
  '(BLUE - RED) / GREEN',
  {
    'BLUE': median.select('B2'),
    'GREEN': median.select('B3'),
    'RED': median.select('B4')
  }
).rename('Chlorophyll_a');

var chlorophyll_clipped = chlorophyll.clip(lake);

Map.addLayer(chlorophyll_clipped, { min: -1, max: 1, palette: ['blue', 'cyan', 'green', 'yellow', 'red'] }, 'Estimated Chlorophyll-a');

var chlStats = chlorophyll_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake,
  scale: 10,
  maxPixels: 1e13
});

print('Mean Chlorophyll-a:', chlStats);
```

# Step 14
Add a Chlorophyll-a Legend.

```js
var chlLegend = ui.Panel({ style: { position: 'top-left', padding: '8px 15px' } });
var chlTitle = ui.Label({ value: 'Chlorophyll-a', style: { fontWeight: 'bold', fontSize: '16px' } });
chlLegend.add(chlTitle);

chlLegend.add(makeRow('blue', 'Low'));
chlLegend.add(makeRow('cyan', 'Moderate'));
chlLegend.add(makeRow('green', 'Elevated'));
chlLegend.add(makeRow('yellow', 'High'));
chlLegend.add(makeRow('red', 'Very High'));

Map.add(chlLegend);
```
![ca.png](..%2FImages%2Fca.png)
# Step 15
Estimate Electrical Conductivity (EC).
```js
var conductivity = median.expression(
  '(BLUE * RED) / GREEN',
  {
    'BLUE': median.select('B2'),
    'GREEN': median.select('B3'),
    'RED': median.select('B4')
  }
).rename('Electrical_Conductivity');

var conductivity_clipped = conductivity.clip(lake);

Map.addLayer(conductivity_clipped, { min: 0, max: 1.5, palette: ['blue', 'cyan', 'green', 'yellow', 'orange', 'red'] }, 'Electrical Conductivity');

var ecStats = conductivity_clipped.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean Electrical Conductivity:', ecStats);
```

# Step 16
Add an EC Legend.

```js
var ecLegend = ui.Panel({ style: { position: 'bottom-center', padding: '8px 15px' } });
var ecTitle = ui.Label({ value: 'Electrical Conductivity', style: { fontWeight: 'bold', fontSize: '16px' } });
ecLegend.add(ecTitle);

ecLegend.add(makeRow('blue', 'Very Low'));
ecLegend.add(makeRow('cyan', 'Low'));
ecLegend.add(makeRow('green', 'Moderate'));
ecLegend.add(makeRow('yellow', 'Elevated'));
ecLegend.add(makeRow('orange', 'High'));
ecLegend.add(makeRow('red', 'Very High'));

Map.add(ecLegend);
```
![ec.png](..%2FImages%2Fec.png)
# Step 17
Map Aquatic Vegetation (NDRE: Normalized Difference Red Edge), use NDRE over NDVI because it better visualizes chlorophyll content in plants.
```js
var ndre = median.expression(
  '(NIR - REDEDGE) / (NIR + REDEDGE)',
  {
    'NIR': median.select('B8'),
    'REDEDGE': median.select('B5')
  }
).rename('NDRE')
  .clip(lake);

Map.addLayer(ndre, { min: -1, max: 1, palette: ['cyan', 'yellow', 'green', 'darkgreen', 'brown'] }, 'Aquatic Vegetation');

var vegStats = ndre.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: lake.geometry(),
  scale: 10,
  maxPixels: 1e13
});

print('Mean Aquatic Vegetation (NDRE):', vegStats);
```

# Step 18
Add an Aquatic Vegetation Legend.

```js
var vegLegend = ui.Panel({ style: { position: 'top-center', padding: '8px 15px' } });
var vegTitle = ui.Label({ value: 'Aquatic Vegetation (NDRE)', style: { fontWeight: 'bold', fontSize: '16px' } });
vegLegend.add(vegTitle);

vegLegend.add(makeRow('yellow', 'Sparse'));
vegLegend.add(makeRow('green', 'Moderate'));
vegLegend.add(makeRow('darkgreen', 'Dense'));
vegLegend.add(makeRow('brown', 'Very Dense'));

Map.add(vegLegend);
```
![av.png](..%2FImages%2Fav.png)

## Summary: Water Quality Parameters

| Parameter | Index / Method | Key Bands | Output Range |
|---|---|---|---|
| **pH** | Regression equation | B2, B4, NDWI, AWEInsh | 5–9 (pH units) |
| **Dissolved Oxygen** | Multi-band regression | B2, B3, B4, B8, B11, B12 | 0–14 mg/L |
| **Turbidity** | Normalized Red-Green | B3, B4 | -1 to 1 (relative) |
| **Chlorophyll-a** | Blue-Red-Green ratio | B2, B3, B4 | -1 to 1 (relative) |
| **Electrical Conductivity** | Blue-Red-Green ratio | B2, B3, B4 | 0–1.5 (relative) |
| **Aquatic Vegetation** | NDRE | B5, B8 | -1 to 1 (relative) |

> **Note:** pH and DO are calibrated equations with physical units. Turbidity, chlorophyll-a, electrical conductivity, and NDRE are spectral indices — they are relative measures useful for spatial comparison but require field calibration to convert to absolute physical units.
