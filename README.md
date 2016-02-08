<!-- MarkdownTOC -->

- [Introduction][introduction]
- [GIS Steps & Notes][gis steps  notes]
    - [County Parcel data][county parcel data]
    - [County Land Use Codes][county land use codes]
    - [City Zoning Data][city zoning data]
        - [Determining Projection][determining projection]
        - [Unique values in `NEW_ZONE` column][unique values in new_zone column]
        - [Adding full names to attribute column in QGIS][adding full names to attribute column in qgis]
        - [Delete unneeded columns][delete unneeded columns]
        - [Export separate layers as GeoJSON][export separate layers as geojson]
- [Understanding City Zoning Data][understanding city zoning data]
    - [Decoding values][decoding values]
    - [Codes for Use Regulations][codes for use regulations]
- [Miscellaneous][miscellaneous]
    - [OpenCounter Santa Cruz][opencounter santa cruz]
        - [Business Types][business types]
        - [Operations Section][operations section]
        - [Team Section][team section]
        - [Site Section][site section]
        - [Summary][summary]
    - [Definitions in Dayton Zoning PDF][definitions in dayton zoning pdf]
        - [Non-Conforming Usage][non-conforming usage]
        - [Retail][retail]
    - [Conditional][conditional]
        - [Conditional Related to Distance][conditional related to distance]

<!-- /MarkdownTOC -->
# Introduction
The goal of this project is to prepare and visualize a zoning shapefile in order to encourage and streamline local business and community initiatives.

Currently, each zone is represented by an individual `.GeoJSON` layer that can be viewed individually by clicking on any of the layers [here](https://github.com/Tele-Pet/zoning/tree/master/GeoJSON_lyrs), or all layers at once [here](https://github.com/Tele-Pet/zoning/blob/master/GeoJSON_lyrs/zoning_4326.geojson).

A rough draft of a custom Leaflet-based map has been started [here](https://github.com/Tele-Pet/zoning/tree/master/qgis_to_leaflet/export_2015_09_06_10_35_09).  It can be downloaded and viewed locally in browser by opening the `index.html` file.

Existing documentation detailing zoning data is available [here](http://www.cityofdayton.org/departments/pcd/planning/Pages/ZoningMap.aspx), and will need to be researched in order to further refine any data visualizations.


# GIS Steps & Notes
## County Parcel data
`Geodatabase_County.zip` can be downloaded from [mcauditor.org](http://www.mcauditor.org/downloads/gis_download_geodb.cfm), from which the `mc_parcel_polygon_e any` layer can be attained.  The following expression can be used to select just those parcels that are part of the city of Dayton:

```
"LOC_AREA"  =  'DAYTON' 
```

**Initially** there appears to not be a column that refers _land use codes_ that are discussed below.

## County Land Use Codes
A list of **land use codes** can be downloaded in PDF from from [mcrealestate.org](http://www.mcrealestate.org/pdffiles/luc.pdf).  (For a list of the other GIS data available, go [here](http://www.mcauditor.org/downloads/gis_filedownload.cfm)) **Initially** there appears to not be any mention of [Non-Conforming Usage][non-conforming usage] that are discussed below.  

## City Zoning Data
### Determining Projection
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

### Unique values in `NEW_ZONE` column
Presuming environment is **set up to run QGIS modules externally**, [getUniqueAttributes_v1.py](google.com) can be run to print out total number of unique values in `NEW_ZONE` column, and a list of all unique names.  Here are the **26** unique values found in [zoning_aug62015.shp](google.com) file: 

```
[NULL, u'AP', u'BP', u'CBD', u'CI', u'EGC', u'EMF', u'ENC', u'ER-3', u'ER-4', u'I-1', u'I-2', u'MGC', u'MH', u'MMF', u'MNC', u'MR-5', u'OS', u'SGC', u'SMF', u'SNC', u'SR-1', u'SR-2', u'T', u'UBD', u'WO']
```

### Adding full names to attribute column in QGIS
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

### Delete unneeded columns
Toggle _on_ edit mode and delete all columns except `zone_name`.  Then, choose _Save as_ option, specifying `.GeoJSON` format and `EPSG:4326` CRS.

### Export separate layers as GeoJSON
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

# Understanding City Zoning Data
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

# Miscellaneous

## OpenCounter Santa Cruz
[OpenCounter Santa Cruz](http://opencounter.cityofsantacruz.com) is an example of a website that...
> will guide you through the process of registering a new business in Santa Cruz, CA.

The site asks the user what type of business they will be running, which includes a graphic interface of _Common Business Types_, as well as a full list of all options.

### Business Types
#### Common Business Types
- Industrial Bakery
- Bar
- Café
- Restaurant
- Hotel/Motel
- Office
- Clothing Store
- Pharmacy
- Bakery
- Preschool
- Pool Hall
- Golf Course
- Gym

#### All Business Types
There is also a [hyperlink option](http://opencounter.cityofsantacruz.com/en/pages/business-type) to view all business types, which include the following:



### Operations Section  

In the [Operations Section](http://opencounter.cityofsantacruz.com/en/sections/operations), there are the following questions:

- What is the extent of the project that you're proposing?
    + Are you planning on making any physical alterations, beyond paint and cosmetic changes to your proposed location? Are you planning a remodel or building project?
- Will the building use change significantly in with this project?
- Does your business name include your last name?
- Are you planning on having live music or events?        
- Do you have plans to install a sign?

Also asks for these addtional details:

- Suite Number
- Number of employees you plan to have by end of first year
- Structure of business
    + Sole Proprietorship
    + General Partnership
    + Limited Partnership
    + Limited Liability Partnership
    + Limited Liability Company
    + Corporation
    + Trust
    + Joint Venture
    + Husband and Wife
    + State of Local Registered Domestic Partners
- Business Phone Number

### Team Section
- Do you have a co-founder?
- Are you working with an architect, engineer or design professional? 
- Do you own the building that you' re proposing for this project?  

Additionally, known information about the owner of the property is requested.

### Site Section
> This section will cover topics that relate to your physical footprint. This includes building permit process and potential development impacts as well as stepping through signage and other external features on site.

> Please note that submitting any information in this section will not submit permit applications to the City.

The [Site Section](http://opencounter.cityofsantacruz.com/en/sections/site) asks the following questions:

- Is your project 100% new construction?
- Does the project require demolition of an existing structure?
- Are you planning on altering existing plumbing features or adding new ones?
- Are you planning on altering existing mechanical features or adding new ones?Are you planning on altering existing electrical features or adding new ones?
- Does the project impact require grading or a new foundation?
- Are you temporarily blocking off the sidewalk (or working above it) during construction?
- Does your location have fire sprinklers?
- Does your building project or planned operations include hazardous materials or substances?
- Is your location ADA accessible? (Americans with Disability Act)
- Will you be cutting into the sidewalk or street to install utilities as part this project?


### Summary
This seciton provides a summary view of all information, and directs the user to the Santa Cruz Planning Department:

> Thank you for your interest in starting a business in Santa Cruz! Please contact the Planning Department with any questions regarding permits to start the business process.

A submit button is also provided, and presumably this leads to followup with all parties.

## Definitions in Dayton Zoning PDF

### Non-Conforming Usage
Non-conforming usage can mean a parcel within a given zone may have been grandfathered into a zone despite not fullfilling a given zone's requirements, and thus can be referred to as a _non-conforming_ parcel.  On page 51, there is further explanation:
> non-conforming uses, buildings, lots, and structures are subject to regulations limiting their use, restoration, reconstruction, extension, and substitution.


### Retail
On page 83, **Retail** is defined as the following:

> An establishment engaged in the selling or renting of goods or merchandise to the general public for personal or household consumption, and rendering services incidental to the sale of such products. Such an establishment is open to the general public during regular business hours and has display areas that are designed and laid out to attract the general public. In determining a use to be a retail use, the proportion of display area vs. storage area and the proportion of the building facade devoted to display windows may be considered. This term does not include any adult entertainment uses. This term includes, but is not limited to, artist’s studios, dry cleaning establishments, laundromats, portrait studios, and bakeries.

## Conditional
### Conditional Related to Distance
On page 313 of Dayton's zoning documentation, the following is stated:

> Any building or other facility, on a zoning lot, used to house or exercise livestock shall be at least 200 feet from any lot in a residential zoning district.

This might be an opportunity to build a query based on spatial analysis-- i.e., a buffer of all areas that are >= 200 feet from a residential zone district.

<!-- Footnotes -->
[^fn1]: [NameMangler](http://manytricks.com/namemangler/) OSX application used to rename files.
