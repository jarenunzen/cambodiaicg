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

//----------------------------------------------------------- 
// VARIABLES OF INTEREST
//----------------------------------------------------------- 
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

// Elevation from NASADEM 
var elevation = ee.Image('NASA/NASADEM_HGT/001').select('elevation').clip(roi);
//var elevation10m = elevation.resample('bilinear').reproject({crs: 'EPSG:4326', scale: 10});

// Slope calculation from here: https://github.com/macleidivarnier/Landslide-susceptibility-mapping-GEE/blob/main/LSM_code_GEE
var slope = ee.Terrain.slope(elevation).reproject({crs: 'EPSG:4326', scale: 10});
//var slope_suitable = slope.updateMask(slope.lte(15));

// Precipitation (CHIRPS - mean annual in mm/year)
var chirps = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var meanAnnualPrecip = chirps.select('precipitation')
  .mean().multiply(365)
  .clip(roi).rename('MeanAnnualPrecip');

// Soil pH (OpenLandMap - scaled x10, so pH 5.5 = 55)
var soilPH = ee.Image('OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02')
  .select('b0') // 0-5cm depth
  .clip(roi).rename('SoilPH');

// Fire Frequency (FIRMS MODIS)
var firms = ee.ImageCollection('FIRMS')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var fireFrequency = firms.select('T21')
  .map(function(img) { return img.gt(0).rename('fire'); })
  .sum()
  .clip(roi).rename('FireFrequency');
  
  
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

```
