Flood Mapping and Monitoring in Nigeria
Introduction

Flooding is a major environmental hazard that causes significant destruction to infrastructure, properties, and livelihoods, especially in low-lying regions like the Niger Delta in Nigeria. Understanding and monitoring flood patterns in these areas is essential for mitigation and disaster management. This project, Flood Mapping and Monitoring in Nigeria, leverages Sentinel-1 Synthetic Aperture Radar (SAR) data and Google Earth Engine (GEE) to monitor and map flood-prone areas in the Niger Delta. The repository contains a script for analyzing Sentinel-1 data to identify flooded and non-flooded areas, calculate flood statistics, and export results for further analysis and decision-making.
Project Setup

To get started with the project, follow these steps:

    Clone the repository:

    bash

git clone git@github.com:eteh1/Flood-Mapping-and-Monitoring-in-Nigeria.git

Navigate to the project directory:

bash

cd Flood-Mapping-and-Monitoring-in-Nigeria

Initialize the Git repository and add the README.md file:

bash

git init
git add README.md
git commit -m "first commit"

Set up the main branch:

bash

git branch -M main

Add the remote repository:

bash

git remote add origin git@github.com:eteh1/Flood-Mapping-and-Monitoring-in-Nigeria.git

Push the initial commit to GitHub:

bash

    git push -u origin main

Flood Mapping using Sentinel-1 Data
Defining the Area of Interest (AOI)

The project focuses on the Niger Delta, an area highly susceptible to flooding due to its geographic location and frequent heavy rainfall. The region is defined using the following asset in Google Earth Engine:

javascript

var aoi = projects/ee-desmondeteh/assets/Niger_Delta_;

Time Period for Analysis

Flooding events are often seasonal or occur after significant weather events. In this script, the analysis covers the period from September 20, 2022, to October 30, 2022, which is within the flood-prone season in Nigeria:

javascript

var startDate = '2022-09-20';
var endDate = '2022-10-30';

Loading Sentinel-1 SAR Dataset

Sentinel-1 SAR imagery is particularly useful for flood mapping due to its ability to penetrate cloud cover and capture data even during adverse weather conditions. The dataset is filtered to obtain images from descending orbits with VV polarization at a resolution of 10 meters:

javascript

var sentinel1 = ee.ImageCollection('COPERNICUS/S1_GRD')
                  .filterBounds(aoi)
                  .filterDate(startDate, endDate)
                  .filter(ee.Filter.eq('instrumentMode', 'IW'))
                  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
                  .filter(ee.Filter.eq('resolution_meters', 10))
                  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
                  .select('VV');

Creating a Median Composite Image

A median composite of the Sentinel-1 images is created to reduce noise and better identify flood-affected areas during the specified period:

javascript

var medianImage = sentinel1.median().clip(aoi);

Identifying Flooded and Non-Flooded Areas

Flooded areas are identified using a threshold value based on backscatter values in the SAR data. Areas with values below the threshold are classified as flooded:

javascript

var floodThreshold = -11;
var floodedArea = medianImage.lt(floodThreshold).selfMask();
var nonFloodedArea = medianImage.gte(floodThreshold).selfMask();

This threshold is adjustable and can be refined based on the characteristics of the area of interest and historical flood data.
Visualization

The results are displayed on the map, with flooded areas marked in red and non-flooded areas marked in blue:

javascript

Map.centerObject(aoi, 10);
Map.addLayer(floodedArea, {palette: ['red'], min: 0, max: 1}, 'Flooded Area');
Map.addLayer(nonFloodedArea, {palette: ['blue'], min: 0, max: 1}, 'Non-Flooded Area');

Flood Area Statistics

To quantify the impact of flooding, the total flooded area is calculated in square meters by multiplying the identified flooded pixels by their area. The results are printed for analysis:

javascript

var floodArea = floodedArea.multiply(ee.Image.pixelArea()).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: aoi,
  scale: 10,
  crs: 'EPSG:4326',
  maxPixels: 1e10
});

var nonFloodArea = nonFloodedArea.multiply(ee.Image.pixelArea()).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: aoi,
  scale: 10,
  crs: 'EPSG:4326',
  maxPixels: 1e10
});

print('Flooded Area (m^2): ', floodArea.get('VV'));
print('Non-Flooded Area (m^2): ', nonFloodArea.get('VV'));

Visualizing the Data

A simple pie chart is generated to provide a visual representation of the ratio between flooded and non-flooded areas:

javascript

var chart = ui.Chart.array.values({
  array: [floodArea.get('VV'), nonFloodArea.get('VV')],
  axis: 0
})
.setChartType('PieChart')
.setOptions({
  title: 'Flood vs Non-Flood Areas',
  slices: [{color: 'red'}, {color: 'blue'}],
  labels: ['Flooded Area', 'Non-Flooded Area']
});

print(chart);

Exporting the Results

The results of the flood mapping are exported as GeoTIFF files for further analysis or integration into GIS platforms. The files can be used to create flood hazard maps or for use in early warning systems:

javascript

Export.image.toDrive({
  image: floodedArea,
  description: 'FloodedArea_TIFF',
  scale: 100,
  region: aoi,
  crs: 'EPSG:4326',
  maxPixels: 1e8,
  fileFormat: 'GeoTIFF'
});

Export.image.toDrive({
  image: nonFloodedArea,
  description: 'NonFloodedArea_TIFF',
  scale: 100,
  region: aoi,
  crs: 'EPSG:4326',
  maxPixels: 1e8,
  fileFormat: 'GeoTIFF'
});

Conclusion

The Flood Mapping and Monitoring in Nigeria project utilizes Sentinel-1 SAR data and Google Earth Engine to effectively map and monitor flood events in the Niger Delta region. The code provided helps to identify, visualize, and export flood areas, allowing for better flood management and mitigation strategies. Through this project, we aim to contribute to reducing the devastating impacts of floods on communities by providing timely and actionable flood mapping data.
