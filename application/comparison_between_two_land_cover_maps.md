```javascript
//s https://twitter.com/bzgeo/status/1451198635392208911?s=21
 
/* Comparison of Esri, ESA 10m global land cover products
code snippets borrowed / modified from Samapriya Roy and Google
see: https://samapriya.github.io/awesome-gee-community-datasets/projects/esrilc2020/
see: https://vitorsveg.users.earthengine.app/view/worldcover
last modified: 21 October 2021 | eac0021@uah.edu */

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

var a = require('users/bzgeo/examples:_ancillary/afr_asia');
var esa_lc = ee.ImageCollection("ESA/WorldCover/v100");
var esri_lc = ee.ImageCollection("projects/sat-io/open-datasets/landcover/ESRI_Global-LULC_10m");

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// Functions + dictionaries for defining color palettes and legends

var dict1 = { // ESA WorldCover
"names": ['Tree cover','Shrubland','Grassland','Cropland','Built-up','Bare / sparse vegetation','Snow and ice',
'Open water','Herbaceous wetland','Mangroves','Moss and lichen'],
"colors": ['006400','ffbb22','ffff4c','f096ff','fa0000','b4b4b4','f0f0f0','0064c8','0096a0','00cf75','fae6a0']};
//"colors": ['#358221','ffbb22','ffff4c','f096ff',"#ED022A",'b4b4b4','f0f0f0','#1A5BAB','0096a0','00cf75','fae6a0']};

var dict2 = { // Esri Land Cover
"names": ["Water","Trees","Grass","Flooded Vegetation","Crops","Scrub/Shrub","Built Area","Bare Ground","Snow/Ice","Clouds"],
"colors": ["#1A5BAB","#358221","#A7D282","#87D19E","#FFDB5C","#EECFA8","#ED022A","#EDE9E4","#F2FAFF","#C8C8C8"]};

var legend1 = ui.Panel({style: {position: 'bottom-right', padding: '8px 15px'}});
var legend2 = ui.Panel({style: {position: 'bottom-left', padding: '8px 15px'}});

function add_legend(to_map,panel, dict, title) {
  
var legendTitle = ui.Label({value: title,style: {fontWeight: 'bold',fontSize: '16px',margin: '0 0 4px 0',padding: '0'}});
panel.add(legendTitle);
var loading = ui.Label('Loading legend...', {margin: '2px 0 4px 0'});
panel.add(loading);
var makeRow = function(color, name) {
var colorBox = ui.Label({style: {backgroundColor: color,padding: '8px',margin: '0 0 4px 0'}});
var description = ui.Label({value: name,style: {margin: '0 0 4px 6px'}});
return ui.Panel({widgets: [colorBox, description],layout: ui.Panel.Layout.Flow('horizontal')});};
var palette = dict['colors'];
var names = dict['names'];
loading.style().set('shown', false);
for (var i = 0; i < names.length; i++) {panel.add(makeRow(palette[i], names[i]));}
to_map.add(panel);
}

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// Create left and right maps for swipe functions
var leftMap = ui.Map();
var rightMap = ui.Map();

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// Add data to viewers and add map legends
add_legend(leftMap, legend2, dict2, 'Esri Land Cover 2020'); // add legend to left map
add_legend(rightMap, legend1, dict1, 'ESA WorldCover 2020'); // add legend to right map

leftMap.addLayer(esri_lc.mosaic(), {min:1, max:10, palette:dict2['colors'],opacity:1}, 'ESRI LULC 10m');
leftMap.addLayer(a.bnds_intl_ln,{palette:['white'],opacity:1},"Int'l bounds", true);
rightMap.addLayer(esa_lc.first(), {}, 'ESA_WC_LC_2020');
rightMap.addLayer(a.bnds_intl_ln,{palette:['white'],opacity:1},"Int'l bounds", true);

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// Generate split panel
var splitPanel = ui.SplitPanel({firstPanel:leftMap,secondPanel:rightMap,wipe:true,style: {stretch: 'both'}});
ui.root.widgets().reset([splitPanel]);
var linker = ui.Map.Linker([leftMap, rightMap]);
leftMap.setCenter(8.34, 10.82, 3); // set center for map viewer
//leftMap.setCenter(-89.9109, 15.9478, 8); // set center for map viewer
```

