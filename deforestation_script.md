# Full Code   
Full code for the deforestation script.

```js
// 1 = cashew
// 2 = cassava
// 3 = rubber
// 4 = paddyrice
// 5 = water
// 6 = developed
// 7 = forest
// 8 = openforest

// Landcover type vars
var cashew = 1;
var cassava = 2;
var rubber = 3;
var paddyrice = 4;
var forest = 7;
var openforest = 8;

// Forest masks
var forest_mask_2017 = lc_2017.eq(forest || openforest);
var forest_mask_2018 = lc_2018.eq(forest || openforest);
var forest_mask_2019 = lc_2019.eq(forest || openforest);
var forest_mask_2020 = lc_2020.eq(forest || openforest);
var forest_mask_2021 = lc_2021.eq(forest || openforest);
var forest_mask_2022 = lc_2022.eq(forest || openforest);
var forest_mask_2023 = lc_2023.eq(forest || openforest);
var forest_mask_2024 = lc_2024.eq(forest || openforest);
var forest_mask_2025 = lc_2025.eq(forest || openforest);

// Cashew masks
var cashew_mask_2017 = lc_2017.eq(cashew);
var cashew_mask_2018 = lc_2018.eq(cashew);
var cashew_mask_2019 = lc_2019.eq(cashew);
var cashew_mask_2020 = lc_2020.eq(cashew);
var cashew_mask_2021 = lc_2021.eq(cashew);
var cashew_mask_2022 = lc_2022.eq(cashew);
var cashew_mask_2023 = lc_2023.eq(cashew);
var cashew_mask_2024 = lc_2024.eq(cashew);
var cashew_mask_2025 = lc_2025.eq(cashew);

// Cashew masks
var cassava_mask_2017 = lc_2017.eq(cassava);
var cassava_mask_2018 = lc_2018.eq(cassava);
var cassava_mask_2019 = lc_2019.eq(cassava);
var cassava_mask_2020 = lc_2020.eq(cassava);
var cassava_mask_2021 = lc_2021.eq(cassava);
var cassava_mask_2022 = lc_2022.eq(cassava);
var cassava_mask_2023 = lc_2023.eq(cassava);
var cassava_mask_2024 = lc_2024.eq(cassava);
var cassava_mask_2025 = lc_2025.eq(cassava);

// Rubber masks
var rubber_mask_2017 = lc_2017.eq(rubber);
var rubber_mask_2018 = lc_2018.eq(rubber);
var rubber_mask_2019 = lc_2019.eq(rubber);
var rubber_mask_2020 = lc_2020.eq(rubber);
var rubber_mask_2021 = lc_2021.eq(rubber);
var rubber_mask_2022 = lc_2022.eq(rubber);
var rubber_mask_2023 = lc_2023.eq(rubber);
var rubber_mask_2024 = lc_2024.eq(rubber);
var rubber_mask_2025 = lc_2025.eq(rubber);

// Paddyrice masks
var rice_mask_2017 = lc_2017.eq(paddyrice);
var rice_mask_2018 = lc_2018.eq(paddyrice);
var rice_mask_2019 = lc_2019.eq(paddyrice);
var rice_mask_2020 = lc_2020.eq(paddyrice);
var rice_mask_2021 = lc_2021.eq(paddyrice);
var rice_mask_2022 = lc_2022.eq(paddyrice);
var rice_mask_2023 = lc_2023.eq(paddyrice);
var rice_mask_2024 = lc_2024.eq(paddyrice);
var rice_mask_2025 = lc_2025.eq(paddyrice);


// Define key variables
var roi = ksws;
var pixel_area = ee.Image.pixelArea();

// Cashew
var forest_to_cashew = forest_mask_2017.and(cashew_mask_2018);
var cashew_cropped = forest_to_cashew.updateMask(forest_to_cashew).clip(roi);

// Function to reduce // Gets total area of landcover type
function reducer_total_area (year) {
  // Crop
  var cropped = year.updateMask(year).clip(roi);

  var area = cropped.multiply(pixel_area).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 10,
  maxPixels: 1e13
  });
  
  var hectares= ee.Number(area.get('b1')).divide(10000);
  return hectares;
}

// Function to reduce // Gets area of forest to crop between two years
function reducer_crop (forest, crop) {
  // Change in landcover
  var change = forest.and(crop);
  
  // Crop
  var change_cropped = change.updateMask(change).clip(roi);

  var area = change_cropped.multiply(pixel_area).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 10,
  maxPixels: 1e13
  });
  return [area, change_cropped];
}

// Function to reduce // Gets area of deforestation between two years
function reducer (year1, year2) {
  // Change in landcover
  var deforestation = year1.and(year2.not());
  
  // Crop
  var deforestation_cropped = deforestation.updateMask(deforestation).clip(roi);

  var area = deforestation_cropped.multiply(pixel_area).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 10,
  maxPixels: 1e13
  });
  return [area, deforestation_cropped];
}

// FOREST TO NONFOREST
var area_1 = ee.Number(reducer(forest_mask_2017, forest_mask_2018)[0].get('b1')).divide(10000);
var area_2 = ee.Number(reducer(forest_mask_2018, forest_mask_2019)[0].get('b1')).divide(10000);
var area_3 = ee.Number(reducer(forest_mask_2019, forest_mask_2020)[0].get('b1')).divide(10000);
var area_4 = ee.Number(reducer(forest_mask_2020, forest_mask_2021)[0].get('b1')).divide(10000);
var area_5 = ee.Number(reducer(forest_mask_2021, forest_mask_2022)[0].get('b1')).divide(10000);
var area_6 = ee.Number(reducer(forest_mask_2022, forest_mask_2023)[0].get('b1')).divide(10000);
var area_7 = ee.Number(reducer(forest_mask_2023, forest_mask_2024)[0].get('b1')).divide(10000);
var area_8 = ee.Number(reducer(forest_mask_2024, forest_mask_2025)[0].get('b1')).divide(10000);

// FOREST TO CASHEW
var cashew_area_1 = ee.Number(reducer_crop(forest_mask_2017, cashew_mask_2018)[0].get('b1')).divide(10000);
var cashew_area_2 = ee.Number(reducer_crop(forest_mask_2018, cashew_mask_2019)[0].get('b1')).divide(10000);
var cashew_area_3 = ee.Number(reducer_crop(forest_mask_2019, cashew_mask_2020)[0].get('b1')).divide(10000);
var cashew_area_4 = ee.Number(reducer_crop(forest_mask_2020, cashew_mask_2021)[0].get('b1')).divide(10000);
var cashew_area_5 = ee.Number(reducer_crop(forest_mask_2021, cashew_mask_2022)[0].get('b1')).divide(10000);
var cashew_area_6 = ee.Number(reducer_crop(forest_mask_2022, cashew_mask_2023)[0].get('b1')).divide(10000);
var cashew_area_7 = ee.Number(reducer_crop(forest_mask_2023, cashew_mask_2024)[0].get('b1')).divide(10000);
var cashew_area_8 = ee.Number(reducer_crop(forest_mask_2024, cashew_mask_2025)[0].get('b1')).divide(10000);

// FOREST TO CASSAVA
var cassava_area_1 = ee.Number(reducer_crop(forest_mask_2017, cassava_mask_2018)[0].get('b1')).divide(10000);
var cassava_area_2 = ee.Number(reducer_crop(forest_mask_2018, cassava_mask_2019)[0].get('b1')).divide(10000);
var cassava_area_3 = ee.Number(reducer_crop(forest_mask_2019, cassava_mask_2020)[0].get('b1')).divide(10000);
var cassava_area_4 = ee.Number(reducer_crop(forest_mask_2020, cassava_mask_2021)[0].get('b1')).divide(10000);
var cassava_area_5 = ee.Number(reducer_crop(forest_mask_2021, cassava_mask_2022)[0].get('b1')).divide(10000);
var cassava_area_6 = ee.Number(reducer_crop(forest_mask_2022, cassava_mask_2023)[0].get('b1')).divide(10000);
var cassava_area_7 = ee.Number(reducer_crop(forest_mask_2023, cassava_mask_2024)[0].get('b1')).divide(10000);
var cassava_area_8 = ee.Number(reducer_crop(forest_mask_2024, cassava_mask_2025)[0].get('b1')).divide(10000);

// FOREST TO RUBBER
var rubber_area_1 = ee.Number(reducer_crop(forest_mask_2017, rubber_mask_2018)[0].get('b1')).divide(10000);
var rubber_area_2 = ee.Number(reducer_crop(forest_mask_2018, rubber_mask_2019)[0].get('b1')).divide(10000);
var rubber_area_3 = ee.Number(reducer_crop(forest_mask_2019, rubber_mask_2020)[0].get('b1')).divide(10000);
var rubber_area_4 = ee.Number(reducer_crop(forest_mask_2020, rubber_mask_2021)[0].get('b1')).divide(10000);
var rubber_area_5 = ee.Number(reducer_crop(forest_mask_2021, rubber_mask_2022)[0].get('b1')).divide(10000);
var rubber_area_6 = ee.Number(reducer_crop(forest_mask_2022, rubber_mask_2023)[0].get('b1')).divide(10000);
var rubber_area_7 = ee.Number(reducer_crop(forest_mask_2023, rubber_mask_2024)[0].get('b1')).divide(10000);
var rubber_area_8 = ee.Number(reducer_crop(forest_mask_2024, rubber_mask_2025)[0].get('b1')).divide(10000);

// FOREST TO PADDYRICE
var rice_area_1 = ee.Number(reducer_crop(forest_mask_2017, rice_mask_2018)[0].get('b1')).divide(10000);
var rice_area_2 = ee.Number(reducer_crop(forest_mask_2018, rice_mask_2019)[0].get('b1')).divide(10000);
var rice_area_3 = ee.Number(reducer_crop(forest_mask_2019, rice_mask_2020)[0].get('b1')).divide(10000);
var rice_area_4 = ee.Number(reducer_crop(forest_mask_2020, rice_mask_2021)[0].get('b1')).divide(10000);
var rice_area_5 = ee.Number(reducer_crop(forest_mask_2021, rice_mask_2022)[0].get('b1')).divide(10000);
var rice_area_6 = ee.Number(reducer_crop(forest_mask_2022, rice_mask_2023)[0].get('b1')).divide(10000);
var rice_area_7 = ee.Number(reducer_crop(forest_mask_2023, rice_mask_2024)[0].get('b1')).divide(10000);
var rice_area_8 = ee.Number(reducer_crop(forest_mask_2024, rice_mask_2025)[0].get('b1')).divide(10000);

// PRINT OUTPUT
print('Deforestation in hectares (2017-2018):', area_1);
print('Deforestation in hectares (2018-2019):', area_2);
print('Deforestation in hectares (2019-2020):', area_3);
print('Deforestation in hectares (2020-2021):', area_4);
print('Deforestation in hectares (2021-2022):', area_5);
print('Deforestation in hectares (2022-2023):', area_6);
print('Deforestation in hectares (2023-2024):', area_7);
print('Deforestation in hectares (2024-2025):', area_8);

print('Forest to cashew in hectares (2017-2018):', cashew_area_1);
print('Forest to cashew in hectares (2018-2019):', cashew_area_2);
print('Forest to cashew in hectares (2019-2020):', cashew_area_3);
print('Forest to cashew in hectares (2020-2021):', cashew_area_4);
print('Forest to cashew in hectares (2021-2022):', cashew_area_5);
print('Forest to cashew in hectares (2022-2023):', cashew_area_6);
print('Forest to cashew in hectares (2023-2024):', cashew_area_7);
print('Forest to cashew in hectares (2024-2025):', cashew_area_8);

print('Forest to cassava in hectares (2017-2018):', cassava_area_1);
print('Forest to cassava in hectares (2018-2019):', cassava_area_2);
print('Forest to cassava in hectares (2019-2020):', cassava_area_3);
print('Forest to cassava in hectares (2020-2021):', cassava_area_4);
print('Forest to cassava in hectares (2021-2022):', cassava_area_5);
print('Forest to cassava in hectares (2022-2023):', cassava_area_6);
print('Forest to cassava in hectares (2023-2024):', cassava_area_7);
print('Forest to cassava in hectares (2024-2025):', cassava_area_8);

print('Forest to rubber in hectares (2017-2018):', rubber_area_1);
print('Forest to rubber in hectares (2018-2019):', rubber_area_2);
print('Forest to rubber in hectares (2019-2020):', rubber_area_3);
print('Forest to rubber in hectares (2020-2021):', rubber_area_4);
print('Forest to rubber in hectares (2021-2022):', rubber_area_5);
print('Forest to rubber in hectares (2022-2023):', rubber_area_6);
print('Forest to rubber in hectares (2023-2024):', rubber_area_7);
print('Forest to rubber in hectares (2024-2025):', rubber_area_8);

print('Forest to paddyrice in hectares (2017-2018):', rice_area_1);
print('Forest to paddyrice in hectares (2018-2019):', rice_area_2);
print('Forest to paddyrice in hectares (2019-2020):', rice_area_3);
print('Forest to paddyrice in hectares (2020-2021):', rice_area_4);
print('Forest to paddyrice in hectares (2021-2022):', rice_area_5);
print('Forest to paddyrice in hectares (2022-2023):', rice_area_6);
print('Forest to paddyrice in hectares (2023-2024):', rice_area_7);
print('Forest to paddyrice in hectares (2024-2025):', rice_area_8);

// PURE AREA OUTPUTS
// Forests
print('Forest in hectares 2017:', reducer_total_area(forest_mask_2017));
print('Forest in hectares 2018:', reducer_total_area(forest_mask_2018));
print('Forest in hectares 2019:', reducer_total_area(forest_mask_2019));
print('Forest in hectares 2020:', reducer_total_area(forest_mask_2020));
print('Forest in hectares 2021:', reducer_total_area(forest_mask_2021));
print('Forest in hectares 2022:', reducer_total_area(forest_mask_2022));
print('Forest in hectares 2023:', reducer_total_area(forest_mask_2023));
print('Forest in hectares 2024:', reducer_total_area(forest_mask_2024));
print('Forest in hectares 2025:', reducer_total_area(forest_mask_2025));

// Cashews
print('Cashews in hectares 2017:', reducer_total_area(cashew_mask_2017));
print('Cashews in hectares 2018:', reducer_total_area(cashew_mask_2018));
print('Cashews in hectares 2019:', reducer_total_area(cashew_mask_2019));
print('Cashews in hectares 2020:', reducer_total_area(cashew_mask_2020));
print('Cashews in hectares 2021:', reducer_total_area(cashew_mask_2021));
print('Cashews in hectares 2022:', reducer_total_area(cashew_mask_2022));
print('Cashews in hectares 2023:', reducer_total_area(cashew_mask_2023));
print('Cashews in hectares 2024:', reducer_total_area(cashew_mask_2024));
print('Cashews in hectares 2025:', reducer_total_area(cashew_mask_2025));

// Cassava
print('Cassava in hectares 2017:', reducer_total_area(cassava_mask_2017));
print('Cassava in hectares 2018:', reducer_total_area(cassava_mask_2018));
print('Cassava in hectares 2019:', reducer_total_area(cassava_mask_2019));
print('Cassava in hectares 2020:', reducer_total_area(cassava_mask_2020));
print('Cassava in hectares 2021:', reducer_total_area(cassava_mask_2021));
print('Cassava in hectares 2022:', reducer_total_area(cassava_mask_2022));
print('Cassava in hectares 2023:', reducer_total_area(cassava_mask_2023));
print('Cassava in hectares 2024:', reducer_total_area(cassava_mask_2024));
print('Cassava in hectares 2025:', reducer_total_area(cassava_mask_2025));

// Rubber
print('Rubber in hectares 2017:', reducer_total_area(rubber_mask_2017));
print('Rubber in hectares 2018:', reducer_total_area(rubber_mask_2018));
print('Rubber in hectares 2019:', reducer_total_area(rubber_mask_2019));
print('Rubber in hectares 2020:', reducer_total_area(rubber_mask_2020));
print('Rubber in hectares 2021:', reducer_total_area(rubber_mask_2021));
print('Rubber in hectares 2022:', reducer_total_area(rubber_mask_2022));
print('Rubber in hectares 2023:', reducer_total_area(rubber_mask_2023));
print('Rubber in hectares 2024:', reducer_total_area(rubber_mask_2024));
print('Rubber in hectares 2025:', reducer_total_area(rubber_mask_2025));

// Paddyrice
print('Rice in hectares 2017:', reducer_total_area(rice_mask_2017));
print('Rice in hectares 2018:', reducer_total_area(rice_mask_2018));
print('Rice in hectares 2019:', reducer_total_area(rice_mask_2019));
print('Rice in hectares 2020:', reducer_total_area(rice_mask_2020));
print('Rice in hectares 2021:', reducer_total_area(rice_mask_2021));
print('Rice in hectares 2022:', reducer_total_area(rice_mask_2022));
print('Rice in hectares 2023:', reducer_total_area(rice_mask_2023));
print('Rice in hectares 2024:', reducer_total_area(rice_mask_2024));
print('Rice in hectares 2025:', reducer_total_area(rice_mask_2025));


// Print percent change // 2017-2018
var old_year = forest_mask_2017;
var total_forests = old_year.updateMask(old_year).clip(roi);
var area = total_forests.multiply(pixel_area).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: roi,
  scale: 10,
  maxPixels: 1e13
  });

var total_forest_area = ee.Number(area.get('b1')).divide(10000);
var percent_change = (area_1.divide(total_forest_area));
print('Percent change forests (2017-2018):', percent_change);

// Add to map
Map.addLayer(reducer(forest_mask_2017, forest_mask_2018)[1], {palette:'red'}, 'deforestation');
Map.addLayer(cashew_cropped, {palette:'yellow'}, 'forest to cashews');

// Chart
var data = ee.List(["2017-2018", "2018-2019", "2019-2020", "2020-2021", "2021-2022", "2022-2023", "2023-2024", "2024-2025"]);
var data_2 = ee.List([area_1, area_2, area_3, area_4, area_5, area_6, area_7, area_8]);
var chart = ui.Chart.array.values(data_2, 0, data)
  .setChartType('ColumnChart');
print(chart);


/*
// Playing around landcover map
var cashews_2017 = lc_2017.eq(1);
var cassava_2017 = lc_2017.eq(2);
var rubber_2017 = lc_2017.eq(3);
var paddyrice_2017 = lc_2017.eq(4);
var water_2017 = lc_2017.eq(5);
var developed_2017 = lc_2017.eq(6);
var forests_2017 = lc_2017.eq(7);
var openforests_2017 = lc_2017.eq(8);


Map.addLayer(cashews_2017.updateMask(cashews_2017).clip(roi), {palette:'f94144'}, 'cashew');
Map.addLayer(cassava_2017.updateMask(cassava_2017).clip(roi), {palette:'f3722c'}, 'cassava');
Map.addLayer(rubber_2017.updateMask(rubber_2017).clip(roi), {palette:'f8961e'}, 'rubber');
Map.addLayer(paddyrice_2017.updateMask(paddyrice_2017).clip(roi), {palette:'f9c74f'}, 'paddyrice');
Map.addLayer(water_2017.updateMask(water_2017).clip(roi), {palette:'blue'}, 'water');
Map.addLayer(developed_2017.updateMask(developed_2017).clip(roi), {palette:'577590'}, 'developed');
Map.addLayer(forests_2017.updateMask(forests_2017).clip(roi), {palette:'43aa8b'}, 'forests');
Map.addLayer(openforests_2017.updateMask(openforests_2017).clip(roi), {palette:'90be6d'}, 'openforests');
*/
```