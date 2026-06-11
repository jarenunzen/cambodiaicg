Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/

**Based on the scripts available here:** https://github.com/saraheb3/AgroforestrySuitability_GEE/blob/main/gee_script/White-Oak-Tree-Growth-Suitability


<img width="648" height="684" alt="image" src="https://github.com/user-attachments/assets/bf2deb81-9ea6-4a08-bdb7-6c53db9dabc4" />

# Step 1
Import KSWS shapefile and landcover classification. 

*Landcover is located under 'data/landcover_maps'. The shapefile is located within the 'data/KSWS_Shapefile' folder.* 

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

**Optionally, add the Distance to Water and Distance to Forest Layers to the Map**
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

**Optional visualization parameters for SRTM elevation data**
```js
// Set elevation visualization properties.
var elevationVis = {
  min: 0,
  max: 800,
};

// Set elevation <= 0 as transparent and add to the map.
Map.addLayer(elevation.updateMask(elevation.gt(0)), elevationVis, 'Elevation', false);
```
<img width="1047" height="634" alt="image" src="https://github.com/user-attachments/assets/290f51b1-9cb1-40bc-9d25-7c615e71b94d" />


## 4.3
Calculate slope from the SRTM elevation data.
```js
var slope = ee.Terrain.slope(elevation).reproject({crs: 'EPSG:4326', scale: 10});
```
> [!NOTE]
> Slope calculation reference: https://github.com/macleidivarnier/Landslide-susceptibility-mapping-GEE/blob/main/LSM_code_GEE

**Optional visualization parameters for slope**
```js
// Set slope visualization properties.
var slopeVis = {
  min: 0,
  max: 70,
};
Map.addLayer(slope, slopeVis, 'Slope', false);
```

<img width="1064" height="632" alt="image" src="https://github.com/user-attachments/assets/849044bc-b184-47e2-beed-76b60e87d26a" />

## 4.4
Extract precipitation data from CHIRPS.


> **Source:** Funk, Chris, Pete Peterson, Martin Landsfeld, Diego Pedreros, James Verdin, Shraddhanand Shukla, Gregory Husak, James Rowland, Laura Harrison, Andrew Hoell & Joel Michaelsen. "The climate hazards infrared precipitation with stations-a new environmental record for monitoring extremes". Scientific Data 2, 150066. doi:10.1038/sdata.2015.66 2015.
> (https://developers.google.com/earth-engine/datasets/catalog/UCSB-CHG_CHIRPS_DAILY)
> 5566m resolution
>
> <img width="1560" height="541" alt="image" src="https://github.com/user-attachments/assets/392e003a-a519-483c-ac6d-08385ce534e7" />


```js
// Precipitation (CHIRPS - mean annual in mm/year)
var chirps = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var meanAnnualPrecip = chirps.select('precipitation')
  .mean().multiply(365)
  .clip(roi).rename('MeanAnnualPrecip');
```

**Visualization parameters for annual precipitation**
>[!NOTE]
>Visualizing this layer can take 30-60 seconds or longer.


```js
// Set slope visualization properties.
var precipVis = {
  min: 1800,
  max: 2500,
  palette: ['001137', '0aab1e', 'e7eb05', 'ff4a2d', 'e90000'],
};
Map.addLayer(meanAnnualPrecip, precipVis, 'Annual Precipitation', false)
```
<img width="1091" height="626" alt="image" src="https://github.com/user-attachments/assets/dbeb0601-67d7-4e4e-9f35-d7378d841700" />


## 4.5
Extract soil pH from OpenLandMap.
<img width="1517" height="684" alt="image" src="https://github.com/user-attachments/assets/1bef97f9-ab13-4173-8a4d-50a8d35e3cba" />
> **Source:** Tomislav Hengl. (2018). Soil pH in H2O at 6 standard depths (0, 10, 30, 60, 100 and 200 cm) at 250 m resolution (Version v02) [Data set]. Zenodo. 10.5281/zenodo.1475459


```js
// Soil pH (OpenLandMap - scaled x10, so pH 5.5 = 55)
var soilPH = ee.Image('OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02')
  .select('b0') // 0-5cm depth
  .clip(roi).rename('SoilPH');
```

**Optional Visualization Parameters**
```js
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
```
<img width="1015" height="671" alt="image" src="https://github.com/user-attachments/assets/1a29182b-5bfe-4048-a024-c28a1226d283" />


## 4.6
Extract fire frequency data from NASA FIRMS.


<img width="525" height="70" alt="image" src="https://github.com/user-attachments/assets/1cd1ba23-4078-4e44-a09e-efbe01c72b2f" />

>**Source:** MODIS Collection 6 NRT Hotspot / Active Fire Detections MCD14DL. Available on-line https://earthdata.nasa.gov/firms. doi:10.5067/FIRMS/MODIS/MCD14DL.NRT.006
>(https://developers.google.com/earth-engine/datasets/catalog/FIRMS#description)

```js
// Fire Frequency (FIRMS MODIS)
var firms = ee.ImageCollection('FIRMS')
  .filterDate('2015-01-01', '2024-12-31')
  .filterBounds(roi);
var fireFrequency = firms.select('T21')
  .map(function(img) { return img.gt(0).rename('fire'); })
  .sum()
  .unmask(0)
  .clip(roi).rename('FireFrequency');
```

**Visualize the Fire Frequency Layer**
```js
// Number of fires within each 1km picture during the last 10 years
var firesVis = {
  min: 1,
  max: 10,
  palette: ['red', 'orange', 'yellow'],
};

Map.addLayer(fireFrequency, firesVis, 'Fires (2016-2026)', false);
```


<img width="1047" height="687" alt="image" src="https://github.com/user-attachments/assets/e14eeac8-501d-4ac7-9b57-425410a41ad5" />



**Open and Visualize the Confidence for detecting fires within each pixel**
```js
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
```



# Step 5
Define suitable ranges for each variable. **These will change depending on the importance of each variable and tree species.** 
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
**These layers may still take a few minutes to render.**


>[!NOTE]
> Step #6 and Step #7 will only show the suitable areas based on the defined values above. Users should support these thresholds (definitions of suitability) based on local knowledge or scientific literature. Spend some time comparing the 'suitable' layers (below) with the more complete range of values for each variable above. 

```js
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
```

# Step 7
Visualize each suitability variable by adding each as a layer on the map.
```js
// Add layers to map
Map.addLayer(elevrange,   {}, 'Suitable Elevation', false);
Map.addLayer(sloperange,  {}, 'Suitable Slope', false);
Map.addLayer(preciprange, {}, 'Suitable Precipitation', false);
Map.addLayer(waterrange,   {}, 'Suitable Distance to Water', false);
Map.addLayer(forestrange,   {}, 'Suitable Distance to Forest', false);
Map.addLayer(phrange,     {}, 'Suitable Soil pH', false);
Map.addLayer(firerange,   {}, 'Low Fire Risk', false);
```

<img width="1892" height="559" alt="image" src="https://github.com/user-attachments/assets/132a6cb1-e4f0-4032-9f63-399543481fac" />
**Distance to Forest** 

# Step 8
Score and visualize suitability based on intersection between variables.

>[!IMPORTANT]
> Because the layers have been filtered already, unsuitable regions of the map (red) are likely not being visualized.

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
> This is a computationally demadning task and may take significant time to process.


# Step 9
Export the suitability layer. 

```js
print(requirementScore, 'Forest Planting Suitability Score');

// Export the classified map to Google Drive
Export.image.toDrive({
  image: requirementScore, // Name of the final layer
  description: 'KeoSeima_ReforestationSuitability',  // File name, will be a GeoTiff format
  scale: 10,  // Spatial resolution/pixel size
  maxPixels: 1e13,  
  crs: 'EPSG:32648', // WGS 84/UTM Zone 48N
  region: ksws  
});
```

<img width="708" height="42" alt="image" src="https://github.com/user-attachments/assets/99251562-54a5-4943-a98f-37bc916f7b9b" />



# Step 10 (optional, adjustmet to a weighted overlay function)
If you wish to adjust the input layers based on a function of layer importance, add the following after Step #9:

```js
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
```

# Step 11 (optional) 
Add the land ownership and roads layers to the map as additional context for the landscape. These files are available in the **Training2026 DATA folder.**
> **Source for Roads:** OpenDevelopment Cambodia, and National Committee for Sub
> https://data.opendevelopmentcambodia.net/en/dataset/map-road-railway-network-market-density
>
> **Source for Land Ownership:**
> TBD

<img width="780" height="1166" alt="image" src="https://github.com/user-attachments/assets/eca9a809-313e-47b5-bc35-9ec7f8f19e21" />

```js

```

# Step 12
Add a legend to the map based on the suitability score.
```js
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
```

# Step 13
Calculate the amount of area which corresponds to the highest suitability class (suitability = 7)

>[!CAUTION]
>Processing times for this step can take a long time at this scale and often cause the computation to time out, it is recommended to only include this step for projects focusing on smaller areas. Alternatively, you can export this layer to ArcGIS Pro / QGIS and calculate the class specific area using that software.


```js
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

<img width="389" height="313" alt="image" src="https://github.com/user-attachments/assets/d6a8bb7e-9271-4a9e-a4c9-e7bde4efee4b" />



# Conclusions
Consider what other layers might impact the suitability of a site for forest planting. This could include layers on biodiversity hot spots, species home ranges, historic deforestation (forest loss), population density, settlements, etc. 

**If any new layers are added, or if the suitability thresholds for any of the layers need to be added, return to Step #4 above.**

#
Return to trainings
(https://github.com/jarenunzen/cambodiaicg/tree/main)
