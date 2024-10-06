Flood Mapping and Monitoring in Nigeria

Welcome to the repository for Flood Mapping and Monitoring in Nigeria, a project aimed at using geospatial technology to analyze and visualize flood-prone areas in the Niger Delta, Nigeria. This project leverages Google Earth Engine (GEE) and Sentinel-1 Synthetic Aperture Radar (SAR) data for accurate flood detection and monitoring.
Introduction

Flooding is a major environmental and socio-economic challenge in Nigeria, particularly in the Niger Delta region. This region experiences frequent flooding due to its low-lying terrain, poor drainage systems, and high rainfall during the rainy season. Flooding leads to displacement, destruction of property, and disruption of livelihoods. To mitigate and manage the effects of floods, there is a need for continuous monitoring, early warning systems, and accurate mapping of flood-prone areas.

The Flood Mapping and Monitoring in Nigeria project aims to provide a comprehensive framework for flood detection and mapping using satellite imagery, which can be integrated into disaster management systems to enhance flood prediction, preparedness, and response.
Project Overview

This project uses Sentinel-1 SAR data from the European Space Agency's Copernicus program, processed within the Google Earth Engine (GEE) environment. Sentinel-1's SAR capabilities are especially useful for flood detection as it can capture imagery regardless of cloud cover or light conditions, making it ideal for real-time flood monitoring.
Objectives

    To detect and map flooded areas in the Niger Delta region using Sentinel-1 SAR imagery.
    To calculate the extent of flooded and non-flooded areas.
    To create visualizations that can help policymakers and disaster management agencies understand the flood dynamics.
    To export flood and non-flood data as GeoTIFF files for further analysis.

Key Features

    Flood Detection: Identification of flood-prone areas using SAR data.
    Visualization: Visual representation of flooded vs. non-flooded areas using custom maps and charts.
    Export Capabilities: Export of flood data in GeoTIFF format for use in GIS and other analytical platforms.

Technology Stack

    Google Earth Engine (GEE): A cloud-based platform that allows for large-scale geospatial analysis.
    Sentinel-1 SAR Data: A radar-based satellite dataset that provides high-resolution data for flood detection.
    Git & GitHub: Version control and collaboration.

Instructions for Running the Project

Follow the instructions below to clone the repository, initialize the project, and run the flood detection script using Google Earth Engine.
Step 1: Clone the Repository

bash

git clone git@github.com:eteh1/Flood-Mapping-and-Monitoring-in-Nigeria.git

This command will create a local copy of the repository on your machine.
Step 2: Initialize the Repository and Set Up Git

If this is your first time setting up the project, you'll need to initialize the Git repository.

bash

git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:eteh1/Flood-Mapping-and-Monitoring-in-Nigeria.git
git push -u origin main

Step 3: Define the Area of Interest (AOI)

You can define the Area of Interest (AOI) as follows:

javascript

// Define your Area of Interest (AOI)
var aoi = projects/ee-desmondeteh/assets/Niger_Delta_;

Replace Niger_Delta_ with your specific area of interest (AOI) in the Google Earth Engine environment.
Step 4: Define the Time Period for Analysis

Specify the time period during which you want to analyze flood events:

javascript

var startDate = '2022-09-20';
var endDate = '2022-10-30';

Modify the startDate and endDate variables based on your needs.
Step 5: Load and Filter Sentinel-1 SAR Data

Load Sentinel-1 SAR data from GEE and filter it according to specific criteria:

javascript

var sentinel1 = ee.ImageCollection('COPERNICUS/S1_GRD')
                  .filterBounds(aoi)
                  .filterDate(startDate, endDate)
                  .filter(ee.Filter.eq('instrumentMode', 'IW'))
                  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
                  .filter(ee.Filter.eq('resolution_meters', 10))
                  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
                  .select('VV');

Step 6: Create a Median Composite Image

Create a median composite image for the defined time period:

javascript

var medianImage = sentinel1.median().clip(aoi);

This step ensures that the SAR imagery is aggregated and clipped to the AOI for further analysis.
Step 7: Identify Flooded Areas

Apply a threshold to identify flooded areas. Adjust the floodThreshold variable based on the AOI to improve accuracy:

javascript

var floodThreshold = -11;
var floodedArea = medianImage.lt(floodThreshold).selfMask();
var nonFloodedArea = medianImage.gte(floodThreshold).selfMask();

Step 8: Visualize the Results

Display the flooded and non-flooded areas on the map:

javascript

Map.centerObject(aoi, 10);
Map.addLayer(floodedArea, {palette: ['red'], min: 0, max: 1}, 'Flooded Area');
Map.addLayer(nonFloodedArea, {palette: ['blue'], min: 0, max: 1}, 'Non-Flooded Area');

Step 9: Calculate Flood Area Statistics

You can calculate the total flooded and non-flooded areas in square meters using the following code:

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

Step 10: Visualize Flood Statistics with a Pie Chart

Visualize the flooded and non-flooded areas using a simple pie chart:

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

Step 11: Export Flood and Non-Flood Data

Export the flood and non-flooded areas as GeoTIFF files for further analysis in GIS software:

javascript

// Export the flooded area as a TIFF file
Export.image.toDrive({
  image: floodedArea,
  description: 'FloodedArea_TIFF',
  scale: 100,
  region: aoi,
  crs: 'EPSG:4326',
  maxPixels: 1e8,
  fileFormat: 'GeoTIFF'
});

// Export the non-flooded area as a TIFF file
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

The Flood Mapping and Monitoring in Nigeria project demonstrates the power of remote sensing and geospatial technology in disaster risk management. By using SAR imagery from Sentinel-1, we can identify flood-prone areas in real-time, calculate the extent of the flooding, and provide valuable data to stakeholders for effective disaster response.
https://code.earthengine.google.com/08bca8b64c0894e29cee5117346c22fa

The project can be further expanded by integrating rainfall data, digital elevation models, and socio-economic data to improve flood prediction models. Additionally, the use of machine learning algorithms can help in automating the process and increasing the accuracy of flood detection.

Feel free to fork, contribute, and expand the project to suit other regions or applications.
License

This project is licensed under the MIT License - see the LICENSE file for details.   

Acknowledgements

We would like to acknowledge the use of Google Earth Engine and the European Space Agency's Copernicus program for providing the Sentinel-1 SAR dataset. This project would not have been possible without their contributions to the field of remote sensing and geospatial analysis.
