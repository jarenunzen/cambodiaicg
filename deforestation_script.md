# Full Code   
Full code for the deforestation script.

```js
var forest = 7;
var openforest = 8;
var forest_mask_2017 = lc_2017.eq(forest || openforest);
var forest_mask_2018 = lc_2018.eq(forest || openforest);
var forest_mask_2019 = lc_2019.eq(forest || openforest);
var forest_mask_2020 = lc_2020.eq(forest || openforest);
var forest_mask_2021 = lc_2021.eq(forest || openforest);
var forest_mask_2022 = lc_2022.eq(forest || openforest);
var forest_mask_2023 = lc_2023.eq(forest || openforest);
var forest_mask_2024 = lc_2024.eq(forest || openforest);
var forest_mask_2025 = lc_2025.eq(forest || openforest);
var cashew_mask_2018 = lc_2024.eq(1);

// Define key variables
var roi = ksws;
var pixel_area = ee.Image.pixelArea();

// Cashew
var forest_to_cashew = forest_mask_2017.and(cashew_mask_2018);
var cashew_cropped = forest_to_cashew.updateMask(forest_to_cashew).clip(roi);

// Function to reduce // Gets area of deforestation between two years
function reducer (year1, year2) {
  // Change in landcover
  var deforestation = year1.and(year2.not());
  
  // Crop
  var deforestation_cropped = deforestation.updateMask(deforestation).clip(roi);

  var area = deforestation_cropped.multiply(pixel_area).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 30,
  maxPixels: 1e13
  });
  return [area, deforestation_cropped];
}


// Hectare output
var area_1 = ee.Number(reducer(forest_mask_2017, forest_mask_2018)[0].get('b1')).divide(10000);
var area_2 = ee.Number(reducer(forest_mask_2018, forest_mask_2019)[0].get('b1')).divide(10000);
var area_3 = ee.Number(reducer(forest_mask_2019, forest_mask_2020)[0].get('b1')).divide(10000);
var area_4 = ee.Number(reducer(forest_mask_2020, forest_mask_2021)[0].get('b1')).divide(10000);
var area_5 = ee.Number(reducer(forest_mask_2021, forest_mask_2022)[0].get('b1')).divide(10000);
var area_6 = ee.Number(reducer(forest_mask_2022, forest_mask_2023)[0].get('b1')).divide(10000);
var area_7 = ee.Number(reducer(forest_mask_2023, forest_mask_2024)[0].get('b1')).divide(10000);
var area_8 = ee.Number(reducer(forest_mask_2024, forest_mask_2025)[0].get('b1')).divide(10000);

print('Deforestation in hectares (2017-2018):', area_1);
print('Deforestation in hectares (2018-2019):', area_2);
print('Deforestation in hectares (2019-2020):', area_3);
print('Deforestation in hectares (2020-2021):', area_4);
print('Deforestation in hectares (2021-2022):', area_5);
print('Deforestation in hectares (2022-2023):', area_6);
print('Deforestation in hectares (2023-2024):', area_7);
print('Deforestation in hectares (2024-2025):', area_8);


// Add to map
Map.addLayer(reducer(forest_mask_2017, forest_mask_2018)[1], {palette:'red'}, 'deforestation');
Map.addLayer(cashew_cropped, {palette:'yellow'}, 'forest to cashews');

// Chart
var data = ee.List(["2017-2018", "2018-2019", "2019-2020", "2020-2021", "2021-2022", "2022-2023", "2023-2024", "2024-2025"]);
var data_2 = ee.List([area_1, area_2, area_3, area_4, area_5, area_6, area_7, area_8]);
var chart = ui.Chart.array.values(data_2, 0, data)
  .setChartType('ColumnChart');
print(chart);

// 1 = cashew
// 2 = cassava
// 3 = rubber
// 4 = paddyrice
// 5 = water
// 6 = developed
// 7 = forest
// 8 = openforest
```