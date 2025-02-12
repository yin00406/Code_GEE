# CDL DOWNLOADING

```javascript
//Imports
var GA = ee.FeatureCollection("projects/ee-yin-ars/assets/GA/GA")

//PARAMETER SETTING
var aoi = GA
var aoi_str = "GA"

// for (var year=2019;year<=2020;year++){ // loop
var year = 2022

var CDL = ee.ImageCollection("USDA/NASS/CDL")
                .filterDate(year.toString()+'-01-01', year.toString()+'-12-31')
                .first().select('cropland')

CDL = CDL.where(CDL.neq(1).and(CDL.neq(2)).and(CDL.neq(10)),0)
                    .where(CDL.eq(10), 3) // 1 corn, 2 cotton, 3 peanuts
    
// CDL = CDL.where(CDL.neq(1).and(CDL.neq(5)),0).where(CDL.eq(5), 2) // 1 corn, 2 soybean


// Drive
Export.image.toDrive(
  {
    image: CDL,
    description: 'CDL'+'_'+year.toString()+'_'+aoi_str,
    region: aoi,
    scale: 30,
    maxPixels: 1e13,
    folder: "GEE_Exported_Images_CDL",
    crs: 'EPSG:4326'
  })

// // Cloud
// Export.image.toCloudStorage({
//     image: CDL,
//     description: 'CDL'+'_'+year.toString()+'_'+aoi_str,
//     region: aoi,
//     scale: 30,
//     maxPixels: 1e13,
//     bucket: "early-prediction-mapping",
//     crs: 'EPSG:4326'
// })
```

