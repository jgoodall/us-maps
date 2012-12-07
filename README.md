# Creation process

The maps here were created using the following steps.

## Get raw shape files
 
### Census
Census files have to be downloaded from a browser. This assumes they are downloaded to ~/Downloads. [us-state/zip/county](http://www.census.gov/cgi-bin/geo/shapefiles2010/main) Available: http://www.census.gov/cgi-bin/geo/shapefiles2010/main

#### States
* Open a browser to: https://www.census.gov/cgi-bin/geo/shapefiles2010/main
* Choose 'States (and equivalent)' and 'Submit'
* Choose 'All states in one national file' for 'State and Equivalent (2010)' and click 'Download'

Then unzip the files:

    mkdir -p raw/state && cd raw/state && unzip ~/Downloads/tl_2010_us_state10.zip && cd ../..
    
#### Counties
* Open a browser to: https://www.census.gov/cgi-bin/geo/shapefiles2010/main
* Choose 'Counties (and equivalent)' and 'Submit'
* Choose 'All states in one national file' for 'County and Equivalent (2010)' and click 'Download'

Then unzip the files:

    mkdir -p raw/county && cd raw/county && unzip ~/Downloads/tl_2010_us_county10.zip && cd ../..


#### Zip codes
* Open a browser to: https://www.census.gov/cgi-bin/geo/shapefiles2010/main
* Choose 'Zip Code Tabulation Areas' and 'Submit'
* Choose 'All states in one national file' for '5-Digit ZIP Code Tabulation Area (2010)' and click 'Download'

Then unzip the files:

    mkdir -p raw/zcta5 && cd raw/zcta5 && unzip ~/Downloads/tl_2010_us_zcta510.zip && cd ../..
    
### Medicare

Medicare files can be downloaded directly from [Dartmouth Atlas](http://www.dartmouthatlas.org/)
[hrr/hsa](http://www.dartmouthatlas.org/Tools/Downloads.aspx?tab=35) Available: http://www.dartmouthatlas.org/Tools/Downloads.aspx?tab=35

#### Hospital Referral Regions

    curl -O http://www.dartmouthatlas.org/downloads/geography/hrr_bdry.zip
    mkdir -p raw/hrr && cd raw/hrr && unzip ../../hrr_bdry.zip && cd ../..
    
#### Hospital Service Area

    curl -O http://www.dartmouthatlas.org/downloads/geography/hsa_bdry.zip
    mkdir -p raw/hsa && cd raw/hsa && unzip ../../hsa_bdry.zip && cd ../..

Clean up .zip files as needed.

## Convert to GeoJSON

Convert using [gdal](http://www.gdal.org/). On a Mac, you can install gdal with [homebrew](http://mxcl.github.com/homebrew/): `brew install gdal`.

    ogr2ogr -f "GeoJSON" hsa.json ./raw/hsa/HSA_Bdry.SHP HSA_Bdry
    ogr2ogr -f "GeoJSON" hrr.json ./raw/hrr/HRR_Bdry.SHP HRR_Bdry
    ogr2ogr -f "GeoJSON" state.json ./raw/state/tl_2010_us_state10.shp tl_2010_us_state10
    ogr2ogr -f "GeoJSON" county.json ./raw/county/tl_2010_us_county10.shp tl_2010_us_county10
    ogr2ogr -f "GeoJSON" zcta5.json ./raw/zcta5/tl_2010_us_zcta510.shp tl_2010_us_zcta510
    
Move the GeoJSON files into the geojson directory

    mkdir -p geojson && mv *.json geojson
    
## Convert to TopoJSON

[TopoJSON](https://github.com/mbostock/topojson) is an extension to GeoJSON that encodes toplogy, resulting in much smaller files. [Nodejs](http://nodejs.org/) is required to run the script. Install topojson using `npm install topojson -g`. No simplification is done on these files. None of the files are combined either, although that is possible. ZCTA file fails when trying to use topojson to convert.

    mkdir -p topojson
    topojson --properties 'HRRCITY' --id-property HRRNUM --out ./topojson/hrr.json ./geojson/hrr.json
    topojson --properties 'HSANAME' --id-property HSA93 --out ./topojson/hsa.json ./geojson/hsa.json
    topojson --properties 'NAME10','STUSPS10' --id-property GEOID10 --out ./topojson/state.json ./geojson/state.json
    topojson --properties 'NAME10','NAMELSAD10' --id-property GEOID10 --out ./topojson/county.json ./geojson/county.json
    topojson --properties 'ZCTA5CE10' --id-property GEOID10 --out ./topojson/zcta5.json ./geojson/zcta5.json
