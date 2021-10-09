# Weather variable calculation

## Export hdd

```javascript
/***
 * hdd (heating degree days): sum(temperature - base_temporature)*N, 
 * N is the number of days that the temperature is above the base temperature (65F or 18.3C)
 ***/

// print(tl_2017_us_msa.first())
// print(NLDAS)

Map.addLayer(tl_2017_us_msa)
Map.centerObject(tl_2017_us_msa, 5)

var NLDAS_2010_2018 = NLDAS.filterDate("2010-01-01", "2019-01-01")
var band = ["temperature_mean"]

var all_hdd = ee.List([])

//t loop time
// it will use a lot of time if you use loop time. You can export one file each year
for(var yearStart=2010; yearStart<=2018; yearStart++){
    
    var yearEnd = yearStart+1
    var date_start = ee.Date.fromYMD(yearStart,1,1)
    var date_end = ee.Date.fromYMD(yearEnd,1,1)
    var NLDAS_each_year = NLDAS_2010_2018.filterDate(date_start, date_end)
    // print("NLDAS_each_year", NLDAS_each_year)

    var day_of_year = ee.List.sequence(1, 365, 1)

    var NLDAS_mean = day_of_year.map(function(d){
        return NLDAS_each_year.filter(ee.Filter.calendarRange({
            start: d, 
            end: ee.Number(d), 
            field: 'day_of_year'
        })).reduce(ee.Reducer.mean()) // add "temperature_mean" to perperty
    })
    // print("NLDAS_mean", NLDAS_mean)

    NLDAS_mean = ee.ImageCollection(NLDAS_mean)
    // print("NLDAS_mean", NLDAS_mean)
    
    var NLDAS_mean_fcol = NLDAS_mean.select(band).map(function(img){
        img = img.reproject(NLDAS.first().projection())
        var img_fcol = img.reduceRegions({
          reducer:ee.Reducer.mean(), // add "mean" to property
          tileScale:16,
          collection:tl_2017_us_msa
        })
        return img_fcol
      }) // The structure of NLDAS_mean_fcol: include 365 FeatureCollection. Each FeatureCollection includes all regions.
  
  var NLDAS_mean_fcol_flatten = NLDAS_mean_fcol.flatten()
  
  var pre_hist = NLDAS_mean_fcol_flatten.filterMetadata('mean', "greater_than", 18.3).select(["NAME", "mean"])
  // print(pre_hist.limit(10))
  // print(pre_hist.size())
  
  var hist = pre_hist.aggregate_histogram("NAME")
  
  var hdd = tl_2017_us_msa.map(function(region){
    var name = region.get("NAME")
    var tmp = pre_hist.filterMetadata('NAME', "equals", name)
    tmp = tmp.aggregate_sum("mean")
    var num = hist.get(name)
    var hdd = tmp.subtract(ee.Number(num).multiply(18.3))
    region = region.set({"hdd":hdd, "year":yearStart, "NAME":name})
    return region
  })
  
  all_hdd = all_hdd.add(hdd)
}
var all_hdd_fcol = ee.FeatureCollection(all_hdd)
var all_hdd_fcol_flatten = all_hdd_fcol.flatten()
// print("all_hdd_fcol_flatten", all_hdd_fcol_flatten)

Export.table.toDrive({
      collection: all_hdd_fcol_flatten,
      description: 'all_hdd_fcol_flatten',
      fileFormat: 'CSV',
      selectors: ["hdd", "NAME", "year"]
    })
```

## Export ost

```javascript
/***
 * extract ost (ozone season temperature):  average daily maximum temperature from May to September
 ***/
 
// Map.addLayer(tl_2017_us_msa)
// Map.centerObject(tl_2017_us_msa, 5)

var features = [];
var fromList = ee.FeatureCollection(features);
// print(fromList)

var NLDAS_2010_2018 = NLDAS.filterDate("2010-01-01", "2019-01-01")
// print(NLDAS_2010_2018.size())

var band = ["temperature_max_mean"]

//t loop time ///////////////////////////////////////////////////////////
for(var yearStart=2010; yearStart<=2018; yearStart++){
  
  // var yearEnd = yearStart+1
  var date_start = ee.Date.fromYMD(yearStart,5,1)
  var date_end = ee.Date.fromYMD(yearStart,10,1)
  var NLDAS_each_year = NLDAS_2010_2018.filterDate(date_start, date_end)//.mean()
  // print("NLDAS_each_year", NLDAS_each_year)

  var day_of_year = ee.List.sequence(122, 273, 1) // May to Sept.
      
  var NLDAS_max = day_of_year.map(function(d) {
    return NLDAS_each_year.filter(ee.Filter.calendarRange({
      start: d, 
      end: ee.Number(d), 
      field: 'day_of_year'
    })).reduce(ee.Reducer.max())
  })
  // print("NLDAS_max", NLDAS_max)
  
  var NLDAS_max_mean = ee.ImageCollection(NLDAS_max).reduce(ee.Reducer.mean())
  // print("NLDAS_max_mean", NLDAS_max_mean)
  NLDAS_max_mean = NLDAS_max_mean.reproject(NLDAS.first().projection())
  
  var indicator_mean = NLDAS_max_mean.select(band)
                        .reduceRegions({ collection:tl_2017_us_msa,
                                          reducer:ee.Reducer.mean(),
                                          tileScale:16
                                          }).map(function(feature){
                                              feature = feature.set("year", yearStart)
                                              return feature
                                            })
  // print("indicator_mean", indicator_mean)
  
  fromList = fromList.merge(indicator_mean)
}
// print("fromList", fromList)

Export.table.toDrive({
      collection: fromList,
      description: 'temperature_max_mean',
      fileFormat: 'CSV',
      selectors: ["NAME", "year", "mean"]
    }) 
```

## Export other variables

```javascript
/***
 * extract band value from different ploygons in different year with some simple operation
 ***/
 
// Map.addLayer(tl_2017_us_msa)
// Map.centerObject(tl_2017_us_msa, 5)

//t make an empty FeatureCollection ///////////////////////////////////////////////////////////
var features = [];
var fromList = ee.FeatureCollection(features);
// print(fromList)

var NLDAS_2010_2018 = NLDAS.filterDate("2010-01-01", "2019-01-01")
// print(NLDAS_2010_2018.size())

var band = ["total_precipitation", "wind_v", "wind_u", "pressure", "specific_humidity", "temperature"]

//t loop time ///////////////////////////////////////////////////////////
for(var yearStart=2010; yearStart<=2018; yearStart++){
  
  var yearEnd = yearStart+1
  var date_start = ee.Date.fromYMD(yearStart,1,1)
  var date_end = ee.Date.fromYMD(yearEnd,1,1)
  var NLDAS_each_year = NLDAS_2010_2018.filterDate(date_start, date_end).mean()
  NLDAS_each_year = NLDAS_each_year.reproject(NLDAS.first().projection())
  
  var indicator_mean = NLDAS_each_year.select(band)
                        .reduceRegions({ collection:tl_2017_us_msa,
                                          reducer:ee.Reducer.mean(),
                                          tileScale:16
                                          }).map(function(feature){
                                              feature = feature.set("year", yearStart)
                                              return feature
                                            })
  fromList = fromList.merge(indicator_mean)
}
// print("fromList", fromList)

Export.table.toDrive({
      collection: fromList,
      description: 'pressure',
      fileFormat: 'CSV',
      selectors: ["NAME", "year", "total_precipitation", "wind_v", "wind_u", "pressure", "specific_humidity", "temperature"]
    }) 
```

