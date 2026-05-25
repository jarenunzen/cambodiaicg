Access to the Google Earth Engine Code Editor: 
https://code.earthengine.google.com/

# Step 1   
Import Data for this training including:

- Keo Seima Updated Boundary
- Cashew reference data
- Cassava reference data
- Developed reference data
- Forest reference data
- Open Forest reference data
- Paddy Rice reference data
- Rubber reference data
- Water reference data
- Grasslands reference data

*This information is located within the 'ReferenceData2025Embeddings' folder of this repository.* 

For more information on **how to upload spatial data (assets) to GEE**, please visit: 
(https://developers.google.com/earth-engine/guides/manage_assets)




# Step 2
Refresh the assets tab and import each of the new layers (assets) into a blank script. Rename the layers to:

- ksws
- cashew
- cassava
- developed
- forest
- othercrop
- paddyrice
- rubber
- water
- grassland

![imports.png](..%2Fimports.png)




# Step 3
Enter a new header (using comment syntax) for the script and save it as ‘KSWS_Embeddings_Training_2025’.


![](https://github.com/user-attachments/assets/6722e3cc-9413-4541-8f32-85a02e9ef2e4)

# Optionally
Add each of the reference data assets to the new map and visualize them as individual layers. 
```js
// Import Reference Data Points
Map.addLayer(cashew, {color: 'white'}, 'Cashew Reference Data'); 
Map.addLayer(cassava, {color: 'white'}, 'Cassava Reference Data'); 
Map.addLayer(rubber, {color: 'red'}, 'Rubber Reference Data'); 
Map.addLayer(paddyrice, {color: 'red'}, 'Paddy Rice Reference Data'); 
Map.addLayer(water, {color: 'blue'}, 'Water Reference Data'); 
Map.addLayer(developed, {color: 'grey'}, 'Developed Reference Data'); 
Map.addLayer(forest, {color: 'green'}, 'Forest Reference Data'); 
Map.addLayer(openforest, {color: 'green'}, 'Open Forest Reference Data'); 
Map.addLayer(grassland, {color: 'green'}, 'Grassland Reference Data'); 
```

# Step 4