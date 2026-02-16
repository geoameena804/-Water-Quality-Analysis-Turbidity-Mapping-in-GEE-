# -Water-Quality-Analysis-Turbidity-Mapping-in-GEE-
Turbidity mapping of Gujarat (India) and Sindh (Pakistan) coastal waters using Google Earth Engine and Sentinel-2 data to analyze sediment dynamics and support marine environmental monitoring.
/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var AOI = 
    /* color: #0b4a8b */
    /* shown: false */
    /* displayProperties: [
      {
        "type": "rectangle"
      }
    ] */
    ee.Geometry.Polygon(
        [[[79.55577618233276, 13.658284497046893],
          [79.55577618233276, 12.406018887387871],
          [81.08287579170776, 12.406018887387871],
          [81.08287579170776, 13.658284497046893]]], null, false);
/***** End of imports. If edited, may not auto-convert in the playground. *****/

// Add Sentinel 2 Image Collection and Filter

var sentinelimage = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
  .select(['B3', 'B4', 'B8'])
  .filterDate('2023-01-01', '2023-12-31')
  .filterBounds(AOI)
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
  .median()
  .multiply(0.0001);

//NDWI Calculation

var ndwi = sentinelimage.normalizedDifference(['B3', 'B8']).rename("NDWI")

//create a binary raster (Set a Threshold on NDWI to Detect Water Areas)

var watermask = ndwi.gt(0).rename('watermask')

//NDTI Calculation

var ndti = sentinelimage.normalizedDifference(['B4', 'B3']).rename("NDTI")

//Clip NDTI from water mask

var clipndti = ndti.updateMask(watermask)

Map.addLayer(clipndti.clip(AOI), {
  palette: ['blue','green','yellow','orange','red']
