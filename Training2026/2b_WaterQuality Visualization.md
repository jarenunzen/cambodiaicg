**Find the full script below**

**Metadata about Sentinel 2 (Harmonized) Surface Reflectance Imagery**


(https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED#bands)



```js
/* QUick Visualization of common water quality parameters */

// =========================================================================
// 1. QUERY SENTINEL 2 IMAGERY AND CLIP TO 'lake'
// =========================================================================
var s2Collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(lake)
  .filterDate('2025-11-01', '2026-01-30') 
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20));

print(s2Collection.size(), 'Number of S2 Images Returned')
var clippedImage = s2Collection.median().clip(lake);

// =========================================================================
// 2. CALCULATE INDICES
// =========================================================================
var ndci = clippedImage.normalizedDifference(['B5', 'B4']).rename('NDCI');
var ciGreen = clippedImage.select('B7').divide(clippedImage.select('B3')).subtract(1).rename('CI_green');
var ndwi = clippedImage.normalizedDifference(['B3', 'B8']).rename('NDWI');

// =========================================================================
// 3. BLUE-TO-GREEN VISUALIZATION SCALE FOR ALL INDICES
// =========================================================================
var trueColorVis = {bands: ['B4', 'B3', 'B2'], min: 0, max: 3000, gamma: 1.4};
var blueToGreenPalette = ['#0000ff', '#00a5ff', '#00ffda', '#00ff73', '#00ff00', '#006400'];

var ndciVis    = {min: -0.1, max: 0.4, palette: blueToGreenPalette};
var ciGreenVis = {min: -0.2, max: 2.0, palette: blueToGreenPalette};
var ndwiVis    = {min: -0.1, max: 0.8, palette: blueToGreenPalette};

// =========================================================================
// 4. MAPS & SPLIT PANEL UI
// =========================================================================
var leftMap = ui.Map();
var rightMap = ui.Map();

// Helper function to load all 4 core layers onto a map instance
function addAllLayers(mapInstance) {
  mapInstance.addLayer(clippedImage, trueColorVis, 'True Color', false);
  mapInstance.addLayer(ndwi, ndwiVis, 'NDWI', false);
  mapInstance.addLayer(ndci, ndciVis, 'NDCI', false);
  mapInstance.addLayer(ciGreen, ciGreenVis, 'CI Green', false);
}

// Load the layers onto both maps
addAllLayers(leftMap);
addAllLayers(rightMap);

var linker = ui.Map.Linker([leftMap, rightMap]);
leftMap.centerObject(lake, 12);

var splitPanel = ui.SplitPanel({
  firstPanel: leftMap, secondPanel: rightMap,
  orientation: 'horizontal', wipe: true, style: {stretch: 'both'}
});
ui.root.clear();
ui.root.add(splitPanel);

// =========================================================================
// 5. LEGEND CREATION UI
// =========================================================================
function makeLegend(title, minVal, maxVal) {
  var legendPanel = ui.Panel({
    style: {position: 'bottom-right', padding: '8px 15px', backgroundColor: '#ffffff'}
  });
  
  var legendTitle = ui.Label({
    value: title, style: {fontWeight: 'bold', fontSize: '13px', margin: '0 0 6px 0'}
  });
  legendPanel.add(legendTitle);

  var lon = ee.Image.pixelLonLat().select('longitude');
  var gradient = lon.multiply(0).add(1);
  var colorBar = ui.Thumbnail({
    image: gradient,
    params: {bbox: '0,0,10,10', dimensions: '140x15', format: 'png', palette: blueToGreenPalette},
    style: {stretch: 'horizontal', margin: '0px 0px 4px 0px'}
  });
  legendPanel.add(colorBar);

  var labelPanel = ui.Panel({
    layout: ui.Panel.Layout.flow('horizontal'),
    style: {stretch: 'horizontal'}
  });
  labelPanel.add(ui.Label(minVal, {margin: '0', fontSize: '11px', fontWeight: 'bold'}));
  labelPanel.add(ui.Label(maxVal, {margin: '0', fontSize: '11px', fontWeight: 'bold'}));
  legendPanel.add(labelPanel);
  
  return legendPanel;
}

// Containers for the legends
var leftLegendContainer = ui.Panel({style: {position: 'bottom-left', backgroundColor: '#ffffff'}});
var rightLegendContainer = ui.Panel({style: {position: 'bottom-right', backgroundColor: '#ffffff'}});
leftMap.add(leftLegendContainer);
rightMap.add(rightLegendContainer);

// =========================================================================
// 6. UNIVERSAL LAYER TOGGLE LOGIC
// =========================================================================

// Master list of menu options
var layerOptions = ['True Color', 'NDWI (Water Extent)', 'NDCI (Chlorophyll)', 'CI Green Index'];

function handleLayerSelection(mapInstance, legendContainer, selectedValue) {
  // Turn all layers off at first
  mapInstance.layers().get(0).setShown(false); // True Color
  mapInstance.layers().get(1).setShown(false); // NDWI
  mapInstance.layers().get(2).setShown(false); // NDCI
  mapInstance.layers().get(3).setShown(false); // CI Green
  
  legendContainer.clear();

  // Activate chosen layer and generate the specific legend
  if (selectedValue === 'True Color') {
    mapInstance.layers().get(0).setShown(true);
  } else if (selectedValue === 'NDWI (Water Extent)') {
    mapInstance.layers().get(1).setShown(true);
    legendContainer.add(makeLegend('NDWI (Water)', '-0.1', '0.8'));
  } else if (selectedValue === 'NDCI (Chlorophyll)') {
    mapInstance.layers().get(2).setShown(true);
    legendContainer.add(makeLegend('NDCI (Chl-a)', '-0.1', '0.4'));
  } else if (selectedValue === 'CI Green Index') {
    mapInstance.layers().get(3).setShown(true);
    legendContainer.add(makeLegend('CI Green Index', '-0.2', '2.0'));
  }
}

// ---- LEFT PANEL DROPDOWN ----
var leftSelect = ui.Select({
  items: layerOptions,
  value: 'True Color', // Default layer left side
  onChange: function(selected) {
    handleLayerSelection(leftMap, leftLegendContainer, selected);
  },
  style: {position: 'top-left'}
});
leftMap.add(leftSelect);

// ---- RIGHT PANEL DROPDOWN ----
var rightSelect = ui.Select({
  items: layerOptions,
  value: 'NDCI (Chlorophyll)', // Default layer right side
  onChange: function(selected) {
    handleLayerSelection(rightMap, rightLegendContainer, selected);
  },
  style: {position: 'top-right'}
});
rightMap.add(rightSelect);

// Initialize initial layout visibility states
handleLayerSelection(leftMap, leftLegendContainer, 'True Color');
handleLayerSelection(rightMap, rightLegendContainer, 'NDCI (Chlorophyll)');

print("Total Sentinel-2 scenes matched:", s2Collection.size());

// end
```
