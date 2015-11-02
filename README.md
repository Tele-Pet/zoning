<!-- MarkdownTOC -->

- [Introduction][introduction]
- [GIS Steps & Notes][gis steps  notes]
    - [Determining Projection][determining projection]
    - [Unique values in `NEW_ZONE` column][unique values in new_zone column]
    - [Adding full names to attribute column in QGIS][adding full names to attribute column in qgis]
    - [Deleted unneeded columns][deleted unneeded columns]
    - [Export separate layers as GeoJSON][export separate layers as geojson]
- [Understanding Zoning Data][understanding zoning data]
    - [Decoding values][decoding values]
    - [Codes for Use Regulations][codes for use regulations]

<!-- /MarkdownTOC -->
# Introduction
The goal of this project is to prepare and visualize a zoning shapefile in order to encourage and streamline local business and community initiatives.

Currently, each zone is represented by an individual `.GeoJSON` layer that can be viewed individually by clicking on any of the layers [here](https://github.com/Tele-Pet/zoning/tree/master/GeoJSON_lyrs), or all layers at once [here](https://github.com/Tele-Pet/zoning/blob/master/GeoJSON_lyrs/zoning_4326.geojson).

A rough draft of a custom Leaflet-based map has been started [here](https://github.com/Tele-Pet/zoning/tree/master/qgis_to_leaflet/export_2015_09_06_10_35_09).  It can be downloaded and viewed locally in browser by opening the `index.html` file.  Also, a working draft of the map is being hosted [here](http://chizu.kontele.com/leaflet_test).

Existing documentation detailing zoning data is available [here](http://www.cityofdayton.org/departments/pcd/planning/Pages/ZoningMap.aspx), and will need to be researched in order to further refine any data visualizations.


# GIS Steps & Notes
## Determining Projection
It was 100% clear what projection the original file is in.  Loading `zoning_aug62015.shp` file into QGIS leads to the following _generated_ CRS:

```
**USER:100000
+proj=lcc +lat_1=38.73333333333333 +lat_2=40.03333333333333 +lat_0=38 +lon_0=-82.5 +x_0=600000 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=us-ft +no_defs
```

However, upon importing the included `zoning_aug62015.prj` to the [prj2epsg](http://prj2epsg.org) website, the following results were gotten:

> **Results**
Found a single exact match for the specified search terms:
>**3735** - NAD_1983_StatePlane_Ohio_South_FIPS_3402_Feet

In QGIS, `zoning_aug62015.shp` is **Saved as...**(reprojected) with **CRS:3735** formally designated.

## Unique values in `NEW_ZONE` column
Presuming environment is **set up to run QGIS modules externally**, [getUniqueAttributes_v1.py](google.com) can be run to print out total number of unique values in `NEW_ZONE` column, and a list of all unique names.  Here are the **26** unique values found in [zoning_aug62015.shp](google.com) file: 

```
[NULL, u'AP', u'BP', u'CBD', u'CI', u'EGC', u'EMF', u'ENC', u'ER-3', u'ER-4', u'I-1', u'I-2', u'MGC', u'MH', u'MMF', u'MNC', u'MR-5', u'OS', u'SGC', u'SMF', u'SNC', u'SR-1', u'SR-2', u'T', u'UBD', u'WO']
```

## Adding full names to attribute column in QGIS
In **field calculator**, with **Create new field** checked, create a new field with the follwing settings:
1. **Output field name:** zone_name
2. **Output field type:** Text(string)
3. **Output field width:** 50

Then run the following expression.  Once that has completed, click the **Save edits** button to save changes.

``` SQL
CASE WHEN  "NEW_ZONE" IS  NULL  THEN 'null'
WHEN "NEW_ZONE" = 'AP' THEN 'Airport'
WHEN "NEW_ZONE" = 'BP' THEN 'Business Park'
WHEN "NEW_ZONE" = 'CBD' THEN 'Central Business District'
WHEN "NEW_ZONE" = 'CI' THEN 'Campus Institutional'
WHEN "NEW_ZONE" = 'EGC' THEN 'Eclectic General Commercial'
WHEN "NEW_ZONE" = 'EMF' THEN 'Eclectic Multi-Family'
WHEN "NEW_ZONE" = 'ENC' THEN 'Eclectic Neighborhood Commercial'
WHEN "NEW_ZONE" = 'ER-3' THEN 'Eclectic Residential'
WHEN "NEW_ZONE" = 'ER-4' THEN 'Eclectic Residential'
WHEN "NEW_ZONE" = 'I-1' THEN 'Light Industrial'
WHEN "NEW_ZONE" = 'I-2' THEN 'General Industrial'
WHEN "NEW_ZONE" = 'MGC' THEN 'Mature General Commercial'
WHEN "NEW_ZONE" = 'MH' THEN 'Manufactured Home'
WHEN "NEW_ZONE" = 'MMF' THEN 'Mature Multi-Family'
WHEN "NEW_ZONE" = 'MNC' THEN 'Mature Neighborhood Commercial'
WHEN "NEW_ZONE" = 'MR-5' THEN 'Mature Residential'
WHEN "NEW_ZONE" = 'OS' THEN 'Open Space'
WHEN "NEW_ZONE" = 'SGC' THEN 'Suburban General Commercial'
WHEN "NEW_ZONE" = 'SMF' THEN 'Suburban Multi-Family'
WHEN "NEW_ZONE" = 'SNC' THEN 'Suburban Neighborhood Commercial'
WHEN "NEW_ZONE" = 'SR-1' THEN 'Suburban Residential'
WHEN "NEW_ZONE" = 'SR-2' THEN 'Suburban Residential'
WHEN "NEW_ZONE" = 'T' THEN 'Transitional'
WHEN "NEW_ZONE" = 'UBD' THEN 'Urban Business District'
WHEN "NEW_ZONE" = 'WO' THEN 'Wellhead Operation'
ELSE 'not found'
END
```

## Deleted unneeded columns
Toggle _on_ edit mode and delete all columns except `zone_name`.  Then, choose _Save as_ option, specifying `.GeoJSON` format and `EPSG:4326` CRS.

## Export separate layers as GeoJSON
Use _Split Vector Layer_ tool (Vector → Data Management Tools → Split Vector Layer) to output a layer for each unique attribute/s.  Rename and reformat to minimal names and lower, snake_case.  [^fn1]

Employ Ben Balter's excellent [script](https://gist.github.com/benbalter/5858851) to batch export `.shp` to `GeoJSON` files:

```Ruby
#geojson conversion
function shp2geojson() {
  ogr2ogr -f GeoJSON -t_srs crs:84 "$1.geojson" "$1.shp"
}

#unzip all files in a directory (only if dealing zip files!)
#for var in *.zip; do unzip "$var"; done

#convert all shapefiles
for var in *.shp; do shp2geojson ${var%\.*}; done
```

**Note: `ogr2ogr` is needed.  That and other environment details are highlighted by Balter on his [site](http://ben.balter.com/2013/06/26/how-to-convert-shapefiles-to-geojson-for-use-on-github/). 

# Understanding Zoning Data
## Decoding values
Page 95 of the **[City of Dayton, Ohio Zoning Code](http://www.cityofdayton.org/departments/pcd/planning/Documents/ZoningCode.pdf)** PDF file summarizes the full names of all `NEW_ZONE` values found in [zoning_aug62015.shp](google.com) file.  For quick reference, see **Value** and **Full Name** columns below.  (**Description** and **Misc** columns are for unofficial notes.)

| Value | Full Name | Description | Misc |
|-------|-----------|-------------|------|
| NULL  |           |             |  One instance, appears to be Westwood Elementary, Westwood Park, and some land north of the park    |
| AP    |     Airport      |             |      |
| BP    |     Business Park      |             |      |
| CBD   |  Central Business District         |             |      |
| CI    |      Campus Institutional     |             |      |
| EGC   |    Eclectic General Commercial       |             |      |
| EMF   |     Eclectic Multi-Family      |             |      |
| ENC   |     Eclectic Neighborhood Commercial      |             |      |
| ER-3  |   Eclectic Residential      |             |      |
| ER-4  |   Eclectic Residential        |             |      |
| I-1   |    Light Industrial       |             |      |
| I-2   |        General Industrial   |             |      |
| MGC   |    Mature General Commercial        |             |      |
| MH    |        Manufactured Home   |             |      |
| MMF   |     Mature Multi-Family      |             |      |
| MNC   |        Mature Neighborhood Commercial   |             |      |
| MR-5  |    Mature Residential        |             |      |
| OS    |     Open Space      |             |      |
| SGC   |   Suburban General Commercial        |             |      |
| SMF   |     Suburban Multi-Family      |             |      |
| SNC   |     Suburban Neighborhood Commercial      |             |      |
| SR-1  |    Suburban Residential       |             |      |
| SR-2  |  Suburban Residential         |             |      |
| T     |     Transitional      |             |      |
| UBD   |    Urban Business District       |             |      |
| WO      |   Wellhead Operation        |             |      |

## Codes for Use Regulations
Beginning on page 95, meanings of the following are provided:

* **P:**  _permitted by right as a **principal use**_ **(p. 96)**
* **P*:** Same as above, though _the characteristics of these uses require development standards that are unique to the specific use, and these standards are not common to the uses generally permitted in the District_ **(p. 96)**
* **C:** _A use listed in a Permitted Use Schedule is permitted as a **conditional use** in a District_ **(p. 97)**
* **A:** _permitted as a subordinate building or use when it is clearly incidental to and located on the same zoning lot as the principal building or use_, i.e., **accessory use** **p. 97** 
* **PD:** _Permitted as part of a Planned Development_ **(p. 104)**


<!-- Footnotes -->
[^fn1]: [NameMangler](http://manytricks.com/namemangler/) OSX application used to rename files.
