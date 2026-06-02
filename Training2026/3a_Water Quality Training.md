## The following is based on a guide from NASA Applied Remote Sensing Training Program (ARSET) and data provided by WCS Cambodia. 
Several other trainings are also available for water quality monitoring using remote sensing through NASA ARSET. 

**Objective:**...

**Study Site:** Tonle Sap Lake, a shapefile for this boundary is provided via OpenDevelopment Cambodia
https://data.opendevelopmentmekong.net/dataset/water-bodies-in-cambodia

NASA AREST Training
- (https://www.earthdata.nasa.gov/learn/trainings/monitoring-water-quality-inland-lakes-using-remote-sensing)


# Step 1
Text....

```js
// =============================== 
// 1. DEFINE STUDY AREA 
// =============================== 

// Example lake point (replace with your lake) 
var lake = Prektoal; 

 
Map.centerObject(lake, 11); 
```


# Step 2
Text...

```js
// =============================== 
// 2. CLOUD MASK FUNCTION 
// =============================== 

function maskS2clouds(image) { 
  var qa = image.select('QA60'); 
  // Bits 10 and 11 are clouds and cirrus 
  var cloudBitMask = 1 << 10; 
  var cirrusBitMask = 1 << 11; 

  // Both flags should be zero 
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0) 
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));

  return image.updateMask(mask) 
      .divide(10000) 
      .copyProperties(image, ['system:time_start']); 
} 
```


# Step 3
Text...

```js
// =============================== 
// 3. LOAD SENTINEL-2 IMAGERY 
// =============================== 

var collection = ee.ImageCollection('COPERNICUS/S2_SR') 
  .filterBounds(lake) 
  .filterDate('2019-07-13', '2019-07-15') 
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 50)) 
  .map(maskS2clouds); 

  
// Create median composite 
var median = collection.median(); 
```


# Step 4
Text...

```js
// =============================== 
// 4. CALCULATE WATER INDICES 
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
```


# Step 5
Text...

```js
// =============================== 
// 5. pH ESTIMATION EQUATION 
// =============================== 

// Example coefficients 
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
    'B2': median.select('B2'), 
    'B4': median.select('B4'), 
    'NDWI': ndwi, 
    'a': a, 
    'b': b, 
    'c': c, 
    'd': d, 
    'e': e 
  } 
).rename('pH'); 
```


# Step 6
Text...

```js
// =============================== 
// 6. DISPLAY pH MAP 
// =============================== 

var phVis = { 
  min: 5, 
  max: 9, 
  palette: [ 'red', 'orange', 'yellow', 'green', 'blue'] 
}; 
```


# Step 7
Text...

```js
// =============================== 
// 7. OPTIONAL WATER MASK 
// =============================== 

// Keep only water pixels 
var waterMask = ndwi.gt(-0.2); 
 
var pH_water = pH.updateMask(waterMask); 

Map.addLayer(pH_water, phVis, 'pH Water Only');
```


# Step 8
Text....

```js
// =============================== 
// 8. CREATE LEGEND 
// =============================== 

var legend = ui.Panel({ 
  style: { 
    position: 'bottom-left', 
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

// Add legend entries 
legend.add(makeRow('red', 'pH 5')); 
legend.add(makeRow('orange', 'pH 6')); 
legend.add(makeRow('yellow', 'pH 7')); 
legend.add(makeRow('green', 'pH 8')); 
legend.add(makeRow('blue', 'pH 9')); 

Map.add(legend);
```


# Step 9
Text...

```js
// =============================== 
// 9. PRINT pH STATISTICS 
// =============================== 

var stats = pH_water.reduceRegion({ 
  reducer: ee.Reducer.mean(), 
  geometry: lake.buffer(3000), 
  scale: 10, 
  maxPixels: 1e9 
}); 

print('Mean Lake pH:', stats); 
```


