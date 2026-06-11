# Full Code   
Feel free to copy and paste the full, working code if you encounter any issues with the training.

```js
//----------------------------------------------------------- 
// IMPORT STUDY AREA AND LOAD DATA
//----------------------------------------------------------- 
// Define the study area
var roi = ksws;

// Add study area to the map
Map.addLayer(roi, {color: 'FF0000'}, 'KSWS', false);
// Center the map to study area
Map.centerObject(roi, 10);

// Landcover
var landcover = ee.Image(lulc);
var forests = landcover.eq(6).or(landcover.eq(7)).clip(roi); // Open forest and forest
var water = landcover.eq(4).clip(roi);

// Compute the distance to the nearest targetted class pixel for each pixel in the ROI
var distanceToWater = water.fastDistanceTransform(30).sqrt()
  .multiply(ee.Image.pixelArea().sqrt()) // converts pixel distance to meters
  .clip(roi)
  .rename('WaterDistance');
var distanceToForest = forests.fastDistanceTransform(30).sqrt()
  .multiply(ee.Image.pixelArea().sqrt()) // converts pixel distance to meters
  .clip(roi)
  .rename('ForestDistance');

// Clip the distance image to the ROI
var clippedDistanceToWater = distanceToWater.clip(roi).rename('WaterDistance');
var clippedDistanceToForest = distanceToForest.clip(roi).rename('ForestDistance');

var visParams = {
  min: 0,
  max: 10000,
  palette: ['0000FF', '00FF00', 'FFFF00', 'FF0000'] // Blue, Green, Yellow, Red
};

Map.addLayer(clippedDistanceToWater, visParams, 'Distance to Water', false);

var visParams = {
  min: 0,
  max: 500,
  palette: ['0000FF', '00FF00', 'FFFF00', 'FF0000'] // Blue, Green, Yellow, Red
};
Map.addLayer(clippedDistanceToForest, visParams, 'Distance to Forest', false);

// Elevation from NASADEM 
var elevation = ee.Image('NASA/NASADEM_HGT/001').select('elevation').clip(roi);

// Set elevation visualization properties.
var elevationVis = {
  min: 0,
  max: 800,
};

// Set elevation <= 0 as transparent and add to the map.
Map.addLayer(elevation.updateMask(elevation.gt(0)), elevationVis, 'Elevation', false);

var slope = ee.Terrain.slope(elevation).reproject({crs: 'EPSG:4326', scale: 10});

// Set slope visualization properties.
var slopeVis = {
  min: 0,
  max: 70,
};
Map.addLayer(slope, slopeVis, 'Slope', false);

// Precipitation (CHIRPS - mean annual in mm/year)
var chirps = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var meanAnnualPrecip = chirps.select('precipitation')
  .mean().multiply(365)
  .clip(roi).rename('MeanAnnualPrecip');
  
  // Set slope visualization properties.
var precipVis = {
  min: 1800,
  max: 2500,
  palette: ['001137', '0aab1e', 'e7eb05', 'ff4a2d', 'e90000'],
};
Map.addLayer(meanAnnualPrecip, precipVis, 'Annual Precipitation', false)

// Soil pH (OpenLandMap - scaled x10, so pH 5.5 = 55)
var soilPH = ee.Image('OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02')
  .select('b0') // 0-5cm depth
  .clip(roi).rename('SoilPH');
  
  var visualization = {
  min: 42,
  max: 110,
  palette: [
    'ff0000', 'ff1c00', 'ff3900', 'ff5500', 'ff7100', 'ff8e00',
    'ffaa00', 'ffc600', 'ffe200', 'ffff00', 'e3ff00', 'c7ff00',
    'aaff00', '8eff00', '72ff00', '55ff00', '39ff00', '1dff00',
    '01ff00', '00ff1c', '00ff38', '00ff54', '00ff71', '00ff8d',
    '00ffa9', '00ffc6', '00ffe2', '00fffe', '00e3ff', '00c7ff',
    '00abff', '008fff', '0072ff', '0056ff', '003aff', '001dff',
    '0001ff', '1b00ff', '3800ff', '5400ff',
  ]
};

Map.addLayer(soilPH, visualization, 'OpenLandMap Soil PH', false);

// Fire Frequency (FIRMS MODIS)
var firms = ee.ImageCollection('FIRMS')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var fireFrequency = firms.select('T21')
  .map(function(img) { return img.gt(0).rename('fire'); })
  .sum()
  .unmask(0)
  .clip(roi).rename('FireFrequency');
  
// Number of fires within each 1km picture during the last 10 years
var firesVis = {
  min: 1,
  max: 10,
  palette: ['red', 'orange', 'yellow'],
};

Map.addLayer(fireFrequency, firesVis, 'Fires (2016-2026)', false);

// Map over the collection to get the confidence band, masking out non-fire pixels
var fireConfidence = firms.map(function(img) {
  var fireMask = img.select('T21').gt(0);
  // 'confidence' is the band name for the MODIS confidence percentage
  return img.select('confidence')
            .updateMask(fireMask) // Only look at confidence where a fire actually happened
            .rename('avgConfidence');
}).mean() // Take the average confidence over the 10-year period
  .clip(roi);

// Visualization parameters for 0% to 100% confidence
var confidenceVis = {
  min: 30, // FIRMS usually considers <30% to be low confidence
  max: 100,
  palette: ['blue', 'purple', 'cyan'], // Distinct colors to separate it from the frequency layer
};

// Add the Confidence Layer (set to false so it doesn't overlap immediately)
Map.addLayer(fireConfidence, confidenceVis, 'Average Fire Confidence (%)', false);

//----------------------------------------------------------- 
// SUITABLE RANGES
//----------------------------------------------------------- 
//Elevation
var elevmin = 0;
var elevmax = 1400;

//Slope
var slopemin = 0;
var slopemax = 15; //0-15 good, 15-30 questionable

// Distance to landcover
var watermin = 50;
var watermax = 1000;
var forestmin = 0;
var forestmax = 2000;

//Precipitation
var precipmin = 750;   
var precipmax = 2030;

//pH
var pHmin = 45;    
var pHmax = 68;

// Fires
var firemax = 5;

//----------------------------------------------------------- 
// CLIP RANGES
//----------------------------------------------------------- 
// BLACK = MOST SUITABLE

// Elevation
var elevrange = (elevation.gte(elevmin)).and(elevation.lte(elevmax));

// Slope 
var sloperange = (slope.gte(slopemin)).and(slope.lte(slopemax));

// Distance to water
var waterrange = clippedDistanceToWater.gte(watermin).and(clippedDistanceToWater.lte(watermax));

// Distance to forest
var forestrange = clippedDistanceToForest.gt(forestmin).and(clippedDistanceToForest.lte(forestmax));

// Precipitation 
var preciprange = meanAnnualPrecip.gte(precipmin).and(meanAnnualPrecip.lte(precipmax));

// Soil Ph
var phrange = soilPH.gte(pHmin).and(soilPH.lte(pHmax)); // using x10 scaled values

// Fires
var firerange = fireFrequency.lte(firemax);

// Add layers to map
Map.addLayer(elevrange,   {}, 'Suitable Elevation', false);
Map.addLayer(sloperange,  {}, 'Suitable Slope', false);
Map.addLayer(preciprange, {}, 'Suitable Precipitation', false);
Map.addLayer(waterrange,   {}, 'Suitable Distance to Water', false);
Map.addLayer(forestrange,   {}, 'Suitable Distance to Forest', false);
Map.addLayer(phrange,     {}, 'Suitable Soil pH', false);
Map.addLayer(firerange,   {}, 'Low Fire Risk', false);

//----------------------------------------------------------- 
// SUITABILITY SCORE AND MAP
//----------------------------------------------------------- 
var totalRequirements = 7; 

var requirementScore = elevrange
  .add(sloperange)
  .add(waterrange)
  .add(forestrange)
  .add(preciprange)
  .add(phrange)
  .add(firerange)
  .rename('Suitability Score');

Map.addLayer(requirementScore, {
  min: 0,
  max: totalRequirements,
  palette: ['red', 'orange', 'yellow', 'lightgreen', 'green']
}, 'Reforestation Suitability', true);

print('Forest Planting Suitability Score');
print(requirementScore);

// Export the classified map to Google Drive
Export.image.toDrive({
  image: requirementScore, // Name of the final layer
  description: 'KeoSeima_ReforestationSuitability',  // File name, will be a GeoTiff format
  scale: 10,  // Spatial resolution/pixel size
  maxPixels: 1e13,  
  crs: 'EPSG:32648', // WGS 84/UTM Zone 48N
  region: ksws  
});

// Example implementation of a weighted ranking scheme 
var weightedScore = (elevrange.multiply(0.15))
  .add(sloperange.multiply(0.10))
  .add(waterrange.multiply(0.20))
  .add(forestrange.multiply(0.10))
  .add(preciprange.multiply(0.15))
  .add(phrange.multiply(0.20))
  .add(firerange.multiply(0.10)); // Total weights sum up to 1.0

Map.addLayer(weightedScore, {
  min: 0,
  max: totalRequirements,
  palette: ['red', 'orange', 'yellow', 'lightgreen', 'green']
}, 'Weighted Reforestation Suitability', false);

//----------------------------------------------------------- 
// ADD MAP LEGEND
//----------------------------------------------------------- 

// 1. Set up the main panel container for the legend
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px',
    backgroundColor: 'white'
  }
});

// 2. Create and style the legend title
var legendTitle = ui.Label({
  value: 'Suitability Score',
  style: {
    fontWeight: 'bold',
    fontSize: '16px',
    margin: '0 0 4px 0',
    padding: '0'
  }
});
legend.add(legendTitle);

// 3. Define the categories, descriptions, and corresponding hex colors
// Matches the palette: ['red', 'orange', 'yellow', 'lightgreen', 'green']
var palette = ['#FF0000', '#FFA500', '#FFFF00', '#90EE90', '#008000'];
var names = [
  '0-1: Completely Unsuitable', 
  '2-3: Low Suitability', 
  '4: Moderate Suitability', 
  '5-6: High Suitability', 
  '7: Optimal (All Criteria Met)'
];

// 4. Helper function to construct a styled row for each category
var makeRow = function(color, name) {
  // Create the colored box indicator
  var colorBox = ui.Label({
    style: {
      backgroundColor: color,
      padding: '8px', // Size of the square colored box
      margin: '0 8px 4px 0'
    }
  });

  // Create the descriptive label
  var description = ui.Label({
    value: name,
    style: {margin: '0 0 4px 0'}
  });

  // Return a horizontal panel containing both the box and the description
  return ui.Panel({
    widgets: [colorBox, description],
    layout: ui.Panel.Layout.Flow('horizontal')
  });
};

// 5. Add all the rows to the legend panel
for (var i = 0; i < 5; i++) {
  legend.add(makeRow(palette[i], names[i]));
}

// 6. Print/add the legend panel to the map canvas UI
Map.add(legend);

//----------------------------------------------------------- 
// FUNCTION: CALCULATE OPTIMAL SUITABILITY AREA (HECTARES)
//----------------------------------------------------------- 

/**
 * Calculates the area of optimal pixels within a region of interest.
 * @param {ee.Image} scoreImage - The final suitability score image (0-7).
 * @param {ee.FeatureCollection} geometry - The region boundary (e.g., ksws/roi).
 * @param {number} scale - The spatial resolution for reduction (meters).
 * @return {ee.Number} Area in hectares.
 */
var calculateOptimalArea = function(scoreImage, geometry, scale) {
  // 1. Isolate pixels that meet ALL 7 criteria (Optimal Category)
  var optimalMask = scoreImage.eq(7);
  
  // 2. Multiply the binary mask (1 for optimal, 0 for other) by the actual pixel area in square meters
  var areaImage = optimalMask.multiply(ee.Image.pixelArea());
  
  // 3. Sum all the pixel area values within your study boundary
  var areaSum = areaImage.reduceRegion({
    reducer: ee.Reducer.sum(),
    geometry: geometry,
    scale: scale,          // Matches your export scale (10m)
    maxPixels: 1e13,       // Prevents memory limits on large regions
    tileScale: 4           // Optimizes parallel processing to avoid timeouts
  });
  
  // 4. Extract the area in square meters and convert it to Hectares (1 Hectare = 10,000 m²)
  var areaMeters = ee.Number(areaSum.get('Suitability Score'));
  var areaHectares = areaMeters.divide(10000);
  
  return areaHectares;
};

```
