## The following is based on a guide from NASA Applied Remote Sensing Training Program (ARSET) and data provided by WCS Cambodia. 
Several other trainings are also available for water quality monitoring using remote sensing through NASA ARSET. 

**Objective:** This training focuses on emote sensing observations from  Sentinel-2 for assessing water quality parameters including: pH, dissolved oxygen, chlorophyll-a concentration, electrical conductivity, turbidity, and the presence of aquatic vegetation within inland water bodies. **This training does not specifically rely on a regression analysis using _in situ_ measurements**. For such methods, we suggest adding on either a simple linear or machine learning regression to the end of this script. 

**Study Site:** Tonle Sap Lake, a shapefile for this boundary is provided via OpenDevelopment Cambodia
https://data.opendevelopmentmekong.net/dataset/water-bodies-in-cambodia

NASA AREST Training
- https://www.earthdata.nasa.gov/learn/trainings/monitoring-water-quality-inland-lakes-using-remote-sensing

# Step 0
Import water body shapefile. Rename the shapefile to WaterBodies.
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


**Optional** 
Add the Sentinel-2 median pixel composite to the map as a natural color composite. 
```js
var visualization = {
  bands: ['B4', 'B3', 'B2'],
  min: 0.02879999950528145,
  max: 0.263949990272522,
};


//Display the median composite on the map. 
Map.addLayer(median, visualization, 'Median Pixel Composite', false);
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

>[!TIP]
> To learn more about water chemisty and pH, visit:
>https://www.epa.gov/caddis/ph
>**OR**
>https://www.usgs.gov/water-science-school/science/ph-and-water
 
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


<img width="335" height="185" alt="image" src="https://github.com/user-attachments/assets/bdb34877-5df9-4a7c-973b-fec15329c7be" />



# Step 9
Estimate Dissolved Oxygen (DO).


>[!TIP]
> For more information on water quality and Dissolve Oxygen (DO) visit:
>
> https://www.usgs.gov/water-science-school/science/dissolved-oxygen-and-water
>
>**OR**
>
>https://www.epa.gov/national-aquatic-resource-surveys/indicators-dissolved-oxygen

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


>[!TIP]
> Turbidity and Water: https://www.usgs.gov/water-science-school/science/turbidity-and-water



```js
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
```

# Step 12
Add a Turbidity Legend. Unlike the pH and DO legends which use numeric thresholds, this legend uses qualitative categories since the turbidity index is a relative (unitless) measure rather than a calibrated physical quantity.

```js
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
```
![turbidity.png](..%2FImages%2Fturbidity.png)
> **Tip:** To display the layer better, go into layer settings (the gear icon next to the layer name), go to the range section, click the drop down labeled custom, and select stretch 98%. 
# Step 13
Estimate Chlorophyll-a


>[!TIP]
> For more information on Chlorophyll-a visit: https://www.epa.gov/national-aquatic-resource-surveys/indicators-chlorophyll


```js
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
```

# Step 14
Add a Chlorophyll-a Legend.

```js
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
```
![ca.png](..%2FImages%2Fca.png)
# Step 15
Estimate Electrical Conductivity (EC).


>[!TIP]
> For more information on electrical conductivity visit:
>
>
>https://www.epa.gov/national-aquatic-resource-surveys/indicators-conductivity
>
>
>**OR**
>
>https://www.usgs.gov/water-science-school/science/conductivity-electrical-conductance-and-water

```js
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
```

# Step 16
Add an EC Legend.

```js
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
```
![ec.png](..%2FImages%2Fec.png)
# Step 17
Map Aquatic Vegetation (NDRE: Normalized Difference Red Edge), use NDRE over NDVI because it better visualizes chlorophyll content in plants.
```js
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
```

# Step 18
Add an Aquatic Vegetation Legend.

```js
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
![av.png](..%2FImages%2Fav.png)

# Step 19
Calculate water quality index.
```js
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

// Normalize chlorphyll a 
var chl_score = ee.Image(100) 
  .subtract( 
    chlorophyll.subtract(-0.8) 
      .divide(1.13 - (-0.8)) 
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
```

# Step 20
Create water quality legend.
```js
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
![wqi.png](..%2FImages%2Fwqi.png)
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
