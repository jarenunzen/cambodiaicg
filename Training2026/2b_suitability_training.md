Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/

**Based on the scripts available here:** https://github.com/saraheb3/AgroforestrySuitability_GEE/blob/main/gee_script/White-Oak-Tree-Growth-Suitability


<img width="648" height="684" alt="image" src="https://github.com/user-attachments/assets/bf2deb81-9ea6-4a08-bdb7-6c53db9dabc4" />

# Step 1
Import KSWS shapefile and landcover classification. 

*Landcover is located under 'data/2025_classified_landcover'. The shsapefile is located within the 'data/KSWS_Shapefile' folder.* 

> [!NOTE]
> These shapefiles were also used or created during Section 1 of this training.

# Step 2
Refresh the assets tab and import each of the new layers (assets) into a blank script. Rename the layers to:

- ksws
- lulc

![imports2.png](..%2FImages%2Fimports2.png)

# Step 3
Assign the study area to a variable and add as a layer to the map.
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
```

# Step 4
Define variables of interest necessary for suitability analysis. The variables defined
in this training are good variables to consider, but may change depending on study interest.
## 4.1 
Extract key landcover types including forest and water. Then, compute distance to each class (in meters).
> [!NOTE] 
> For more information on fastDistanceTransform: https://developers.google.com/earth-engine/apidocs/ee-image-fastdistancetransform

```js
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
```

## Optionally, add the Distance to Water and Distance to Forest Layers to the Map
```js
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
```

<img width="1085" height="693" alt="image" src="https://github.com/user-attachments/assets/96fa2ec3-4ea8-42e0-b775-d93e1393a1a0" />


## 4.2
Extract elevation data from NASADEM dataset.

> **Source:** https://developers.google.com/earth-engine/datasets/catalog/NASA_NASADEM_HGT_001
> NASA JPL (2020). NASADEM Merged DEM Global 1 arc second V001 [Data set]. NASA EOSDIS Land Processes DAAC. Accessed 2020-12-30 from doi:10.5067/MEaSUREs/NASADEM/NASADEM_HGT.001
```js
// Elevation from NASADEM 
var elevation = ee.Image('NASA/NASADEM_HGT/001').select('elevation').clip(roi);
```

Optional visualization parameters for SRTM elevation data
```js
// Set elevation visualization properties.
var elevationVis = {
  min: 0,
  max: 800,
};

// Set elevation <= 0 as transparent and add to the map.
Map.addLayer(elevation.updateMask(elevation.gt(0)), elevationVis, 'Elevation', false);
```

## 4.3
Calculate slope from elevation data.
```js
var slope = ee.Terrain.slope(elevation).reproject({crs: 'EPSG:4326', scale: 10});
```
> [!NOTE]
> Slope calculation reference: https://github.com/macleidivarnier/Landslide-susceptibility-mapping-GEE/blob/main/LSM_code_GEE

## 4.4
Extract precipitation data from CHIRPS.
```js
// Precipitation (CHIRPS - mean annual in mm/year)
var chirps = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var meanAnnualPrecip = chirps.select('precipitation')
  .mean().multiply(365)
  .clip(roi).rename('MeanAnnualPrecip');
```

## 4.5
Extract soil pH from OpenLandMap.
```js
// Soil pH (OpenLandMap - scaled x10, so pH 5.5 = 55)
var soilPH = ee.Image('OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02')
  .select('b0') // 0-5cm depth
  .clip(roi).rename('SoilPH');
```

## 4.6
Extract fir frequency data from NASA FIRMS.
```js
// Fire Frequency (FIRMS MODIS)
var firms = ee.ImageCollection('FIRMS')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var fireFrequency = firms.select('T21')
  .map(function(img) { return img.gt(0).rename('fire'); })
  .sum()
  .clip(roi).rename('FireFrequency');
```

# Step 5
Define suitable ranges for each variable. These will change depending on the importance of each variable and tree species. 
For example, if soil pH is extremely important for a particular tree species tha we want to replant, we will define an exact number
to restrict the suitable range to. Otherwise, a more broad range is acceptable.

```js
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
```

# Step 6
Clip each variable to the defined suitable ranges.
```js
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
```

# Step 7
Visualize each suitability variable by adding each as a layer on the map.
```js
// Add layers to map
Map.addLayer(elevrange,   {}, 'Suitable Elevation', false);
Map.addLayer(sloperange,  {}, 'Suitable Slope', false);
Map.addLayer(preciprange, {}, 'Suitable Precipitation', false);
Map.addLayer(phrange,     {}, 'Suitable Soil pH', false);
Map.addLayer(firerange,   {}, 'Low Fire Risk', false);
```

# Step 8
Score and visualize suitability based on intersection between variables.
```js
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
![suitability.png](..%2FImages%2Fsuitability.png)
> [!WARNING]
> This is a computationally expensive task and may take significant time to process.
