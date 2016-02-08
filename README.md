<!-- MarkdownTOC -->

- [nzip all files in a directory (only if dealing zip files!)][nzip all files in a directory only if dealing zip files]
- [or var in *.zip; do unzip &quot;$var&quot;; done][or var in zip do unzip quotvarquot done]
- [onvert all shapefiles][onvert all shapefiles]

<!-- /MarkdownTOC -->

<h1 id="introduction">Introduction</h1>

<p>The goal of this project is to prepare and visualize a zoning shapefile in order to encourage and streamline local business and community initiatives.</p>

<p>Currently, each zone is represented by an individual <code>.GeoJSON</code> layer that can be viewed individually by clicking on any of the layers <a href="https://github.com/Tele-Pet/zoning/tree/master/GeoJSON_lyrs">here</a>, or all layers at once <a href="https://github.com/Tele-Pet/zoning/blob/master/GeoJSON_lyrs/zoning_4326.geojson">here</a>.</p>

<p>A rough draft of a custom Leaflet-based map has been started <a href="https://github.com/Tele-Pet/zoning/tree/master/qgis_to_leaflet/export_2015_09_06_10_35_09">here</a>. It can be downloaded and viewed locally in browser by opening the <code>index.html</code> file.</p>

<p>Existing documentation detailing zoning data is available <a href="http://www.cityofdayton.org/departments/pcd/planning/Pages/ZoningMap.aspx">here</a>, and will need to be researched in order to further refine any data visualizations.</p>

<h1 id="gisstepsnotes">GIS Steps &amp; Notes</h1>

<h2 id="countyparceldata">County Parcel data</h2>

<p><code>Geodatabase_County.zip</code> can be downloaded from <a href="http://www.mcauditor.org/downloads/gis_download_geodb.cfm">mcauditor.org</a>, from which the <code>mc_parcel_polygon_e any</code> layer can be attained. The following expression can be used to select just those parcels that are part of the city of Dayton:</p>
<pre><code class="(null)">&quot;LOC_AREA&quot;  =  &apos;DAYTON&apos; </code></pre>

<p><strong>Initially</strong> there appears to not be a column that refers <em>land use codes</em> that are discussed below.</p>

<h2 id="countylandusecodes">County Land Use Codes</h2>

<p>A list of <strong>land use codes</strong> can be downloaded in PDF from from <a href="http://www.mcrealestate.org/pdffiles/luc.pdf">mcrealestate.org</a>. (For a list of the other GIS data available, go <a href="http://www.mcauditor.org/downloads/gis_filedownload.cfm">here</a>) <strong>Initially</strong> there appears to not be any mention of <a href="#non-conformingusage">Non-Conforming Usage</a> that are discussed below. </p>

<h2 id="cityzoningdata">City Zoning Data</h2>

<h3 id="determiningprojection">Determining Projection</h3>

<p>It was 100% clear what projection the original file is in. Loading <code>zoning_aug62015.shp</code> file into QGIS leads to the following <em>generated</em> CRS:</p>
<pre><code class="(null)">**USER:100000
+proj=lcc +lat_1=38.73333333333333 +lat_2=40.03333333333333 +lat_0=38 +lon_0=-82.5 +x_0=600000 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=us-ft +no_defs</code></pre>

<p>However, upon importing the included <code>zoning_aug62015.prj</code> to the <a href="http://prj2epsg.org">prj2epsg</a> website, the following results were gotten:</p>

<blockquote>
<p><strong>Results</strong><br/>
Found a single exact match for the specified search terms:<br/>
<strong>3735</strong> - NAD_1983_StatePlane_Ohio_South_FIPS_3402_Feet</p>
</blockquote>

<p>In QGIS, <code>zoning_aug62015.shp</code> is <strong>Saved as&#8230;</strong>(reprojected) with <strong>CRS:3735</strong> formally designated.</p>

<h3 id="uniquevaluesinnew_zonecolumn">Unique values in <code>NEW_ZONE</code> column</h3>

<p>Presuming environment is <strong>set up to run QGIS modules externally</strong>, <a href="google.com">getUniqueAttributes_v1.py</a> can be run to print out total number of unique values in <code>NEW_ZONE</code> column, and a list of all unique names. Here are the <strong>26</strong> unique values found in <a href="google.com">zoning_aug62015.shp</a> file: </p>
<pre><code class="(null)">[NULL, u&apos;AP&apos;, u&apos;BP&apos;, u&apos;CBD&apos;, u&apos;CI&apos;, u&apos;EGC&apos;, u&apos;EMF&apos;, u&apos;ENC&apos;, u&apos;ER-3&apos;, u&apos;ER-4&apos;, u&apos;I-1&apos;, u&apos;I-2&apos;, u&apos;MGC&apos;, u&apos;MH&apos;, u&apos;MMF&apos;, u&apos;MNC&apos;, u&apos;MR-5&apos;, u&apos;OS&apos;, u&apos;SGC&apos;, u&apos;SMF&apos;, u&apos;SNC&apos;, u&apos;SR-1&apos;, u&apos;SR-2&apos;, u&apos;T&apos;, u&apos;UBD&apos;, u&apos;WO&apos;]</code></pre>

<h3 id="addingfullnamestoattributecolumninqgis">Adding full names to attribute column in QGIS</h3>

<p>In <strong>field calculator</strong>, with <strong>Create new field</strong> checked, create a new field with the follwing settings:<br/>
1. <strong>Output field name:</strong> zone_name<br/>
2. <strong>Output field type:</strong> Text(string)<br/>
3. <strong>Output field width:</strong> 50</p>

<p>Then run the following expression. Once that has completed, click the <strong>Save edits</strong> button to save changes.</p>
<pre><code class="SQL">CASE WHEN  &quot;NEW_ZONE&quot; IS  NULL  THEN &apos;null&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;AP&apos; THEN &apos;Airport&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;BP&apos; THEN &apos;Business Park&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;CBD&apos; THEN &apos;Central Business District&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;CI&apos; THEN &apos;Campus Institutional&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;EGC&apos; THEN &apos;Eclectic General Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;EMF&apos; THEN &apos;Eclectic Multi-Family&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;ENC&apos; THEN &apos;Eclectic Neighborhood Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;ER-3&apos; THEN &apos;Eclectic Residential&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;ER-4&apos; THEN &apos;Eclectic Residential&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;I-1&apos; THEN &apos;Light Industrial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;I-2&apos; THEN &apos;General Industrial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;MGC&apos; THEN &apos;Mature General Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;MH&apos; THEN &apos;Manufactured Home&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;MMF&apos; THEN &apos;Mature Multi-Family&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;MNC&apos; THEN &apos;Mature Neighborhood Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;MR-5&apos; THEN &apos;Mature Residential&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;OS&apos; THEN &apos;Open Space&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;SGC&apos; THEN &apos;Suburban General Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;SMF&apos; THEN &apos;Suburban Multi-Family&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;SNC&apos; THEN &apos;Suburban Neighborhood Commercial&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;SR-1&apos; THEN &apos;Suburban Residential&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;SR-2&apos; THEN &apos;Suburban Residential&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;T&apos; THEN &apos;Transitional&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;UBD&apos; THEN &apos;Urban Business District&apos;
WHEN &quot;NEW_ZONE&quot; = &apos;WO&apos; THEN &apos;Wellhead Operation&apos;
ELSE &apos;not found&apos;
END</code></pre>

<h3 id="deleteunneededcolumns">Delete unneeded columns</h3>

<p>Toggle <em>on</em> edit mode and delete all columns except <code>zone_name</code>. Then, choose <em>Save as</em> option, specifying <code>.GeoJSON</code> format and <code>EPSG:4326</code> CRS.</p>

<h3 id="exportseparatelayersasgeojson">Export separate layers as GeoJSON</h3>

<p>Use <em>Split Vector Layer</em> tool (Vector → Data Management Tools → Split Vector Layer) to output a layer for each unique attribute/s. Rename and reformat to minimal names and lower, snake_case. <a href="#fn:1" id="fnref:1" title="see footnote" class="footnote">[1]</a></p>

<p>Employ Ben Balter&#8217;s excellent <a href="https://gist.github.com/benbalter/5858851">script</a> to batch export <code>.shp</code> to <code>GeoJSON</code> files:</p>
<pre><code class="Ruby">#geojson conversion
function shp2geojson() {
  ogr2ogr -f GeoJSON -t_srs crs:84 &quot;$1.geojson&quot; &quot;$1.shp&quot;
}

#unzip all files in a directory (only if dealing zip files!)
#for var in *.zip; do unzip &quot;$var&quot;; done

#convert all shapefiles
for var in *.shp; do shp2geojson ${var%\.*}; done</code></pre>

<p>**Note: <code>ogr2ogr</code> is needed. That and other environment details are highlighted by Balter on his <a href="http://ben.balter.com/2013/06/26/how-to-convert-shapefiles-to-geojson-for-use-on-github/">site</a>. </p>

<h1 id="understandingcityzoningdata">Understanding City Zoning Data</h1>

<h2 id="decodingvalues">Decoding values</h2>

<p>Page 95 of the <strong><a href="http://www.cityofdayton.org/departments/pcd/planning/Documents/ZoningCode.pdf">City of Dayton, Ohio Zoning Code</a></strong> PDF file summarizes the full names of all <code>NEW_ZONE</code> values found in <a href="google.com">zoning_aug62015.shp</a> file. For quick reference, see <strong>Value</strong> and <strong>Full Name</strong> columns below. (<strong>Description</strong> and <strong>Misc</strong> columns are for unofficial notes.)</p>

<table>
<colgroup>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
<col style="text-align:left;"/>
</colgroup>

<thead>
<tr>
    <th style="text-align:left;">Value</th>
    <th style="text-align:left;">Full Name</th>
    <th style="text-align:left;">Description</th>
    <th style="text-align:left;">Misc</th>
</tr>
</thead>

<tbody>
<tr>
    <td style="text-align:left;">NULL</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;">One instance, appears to be Westwood Elementary, Westwood Park, and some land north of the park</td>
</tr>
<tr>
    <td style="text-align:left;">AP</td>
    <td style="text-align:left;">Airport</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">BP</td>
    <td style="text-align:left;">Business Park</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">CBD</td>
    <td style="text-align:left;">Central Business District</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">CI</td>
    <td style="text-align:left;">Campus Institutional</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">EGC</td>
    <td style="text-align:left;">Eclectic General Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">EMF</td>
    <td style="text-align:left;">Eclectic Multi-Family</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">ENC</td>
    <td style="text-align:left;">Eclectic Neighborhood Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">ER&#8211;3</td>
    <td style="text-align:left;">Eclectic Residential</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">ER&#8211;4</td>
    <td style="text-align:left;">Eclectic Residential</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">I&#8211;1</td>
    <td style="text-align:left;">Light Industrial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">I&#8211;2</td>
    <td style="text-align:left;">General Industrial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">MGC</td>
    <td style="text-align:left;">Mature General Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">MH</td>
    <td style="text-align:left;">Manufactured Home</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">MMF</td>
    <td style="text-align:left;">Mature Multi-Family</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">MNC</td>
    <td style="text-align:left;">Mature Neighborhood Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">MR&#8211;5</td>
    <td style="text-align:left;">Mature Residential</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">OS</td>
    <td style="text-align:left;">Open Space</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">SGC</td>
    <td style="text-align:left;">Suburban General Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">SMF</td>
    <td style="text-align:left;">Suburban Multi-Family</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">SNC</td>
    <td style="text-align:left;">Suburban Neighborhood Commercial</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">SR&#8211;1</td>
    <td style="text-align:left;">Suburban Residential</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">SR&#8211;2</td>
    <td style="text-align:left;">Suburban Residential</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">T</td>
    <td style="text-align:left;">Transitional</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">UBD</td>
    <td style="text-align:left;">Urban Business District</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
<tr>
    <td style="text-align:left;">WO</td>
    <td style="text-align:left;">Wellhead Operation</td>
    <td style="text-align:left;"></td>
    <td style="text-align:left;"></td>
</tr>
</tbody>
</table>

<h2 id="codesforuseregulations">Codes for Use Regulations</h2>

<p>Beginning on page 95, meanings of the following are provided:</p>

<ul>
<li><strong>P:</strong> <em>permitted by right as a <strong>principal use</strong></em> <strong>(p. 96)</strong></li>
<li><strong>P<em>:</em>* Same as above, though <em>the characteristics of these uses require development standards that are unique to the specific use, and these standards are not common to the uses generally permitted in the District</em> </strong>(p. 96)**</li>
<li><strong>C:</strong> <em>A use listed in a Permitted Use Schedule is permitted as a <strong>conditional use</strong> in a District</em> <strong>(p. 97)</strong></li>
<li><strong>A:</strong> <em>permitted as a subordinate building or use when it is clearly incidental to and located on the same zoning lot as the principal building or use</em>, i.e., <strong>accessory use</strong> <strong>p. 97</strong></li>
<li><strong>PD:</strong> <em>Permitted as part of a Planned Development</em> <strong>(p. 104)</strong></li>
</ul>

<h1 id="miscellaneous">Miscellaneous</h1>

<h2 id="opencountersantacruz">OpenCounter Santa Cruz</h2>

<p><a href="http://opencounter.cityofsantacruz.com">OpenCounter Santa Cruz</a> is an example of a website that&#8230; </p>

<blockquote>
<p>will guide you through the process of registering a new business in Santa Cruz, CA.</p>
</blockquote>

<p>The site asks the user what type of business they will be running, which includes a graphic interface of <em>Common Business Types</em>, as well as a full list of all options.</p>

<h3 id="businesstypes">Business Types</h3>

<h4 id="commonbusinesstypes">Common Business Types</h4>

<ul>
<li>Industrial Bakery</li>
<li>Bar</li>
<li>Café</li>
<li>Restaurant</li>
<li>Hotel/Motel</li>
<li>Office</li>
<li>Clothing Store</li>
<li>Pharmacy</li>
<li>Bakery</li>
<li>Preschool</li>
<li>Pool Hall</li>
<li>Golf Course</li>
<li>Gym</li>
</ul>

<h4 id="allbusinesstypes">All Business Types</h4>

<p>There is also a <a href="http://opencounter.cityofsantacruz.com/en/pages/business-type">hyperlink option</a> to view all business types, which include the following:</p>

<h3 id="operationssection">Operations Section</h3>

<p>In the <a href="http://opencounter.cityofsantacruz.com/en/sections/operations">Operations Section</a>, there are the following questions:</p>

<ul>
<li>What is the extent of the project that you&#8217;re proposing?

<ul>
<li>Are you planning on making any physical alterations, beyond paint and cosmetic changes to your proposed location? Are you planning a remodel or building project?</li>
</ul></li>
<li>Will the building use change significantly in with this project?</li>
<li>Does your business name include your last name?</li>
<li>Are you planning on having live music or events?</li>
<li>Do you have plans to install a sign?</li>
</ul>

<p>Also asks for these addtional details:</p>

<ul>
<li>Suite Number</li>
<li>Number of employees you plan to have by end of first year</li>
<li>Structure of business

<ul>
<li>Sole Proprietorship</li>
<li>General Partnership</li>
<li>Limited Partnership</li>
<li>Limited Liability Partnership</li>
<li>Limited Liability Company</li>
<li>Corporation</li>
<li>Trust</li>
<li>Joint Venture</li>
<li>Husband and Wife</li>
<li>State of Local Registered Domestic Partners</li>
</ul></li>
<li>Business Phone Number</li>
</ul>

<h3 id="teamsection">Team Section</h3>

<ul>
<li>Do you have a co-founder?</li>
<li>Are you working with an architect, engineer or design professional?</li>
<li>Do you own the building that you&#8217; re proposing for this project?</li>
</ul>

<p>Additionally, known information about the owner of the property is requested.</p>

<h3 id="sitesection">Site Section</h3>

<blockquote>
<p>This section will cover topics that relate to your physical footprint. This includes building permit process and potential development impacts as well as stepping through signage and other external features on site.</p>

<p>Please note that submitting any information in this section will not submit permit applications to the City.</p>
</blockquote>

<p>The <a href="http://opencounter.cityofsantacruz.com/en/sections/site">Site Section</a> asks the following questions:</p>

<ul>
<li>Is your project 100% new construction?</li>
<li>Does the project require demolition of an existing structure?</li>
<li>Are you planning on altering existing plumbing features or adding new ones?</li>
<li>Are you planning on altering existing mechanical features or adding new ones?Are you planning on altering existing electrical features or adding new ones?</li>
<li>Does the project impact require grading or a new foundation?</li>
<li>Are you temporarily blocking off the sidewalk (or working above it) during construction?</li>
<li>Does your location have fire sprinklers?</li>
<li>Does your building project or planned operations include hazardous materials or substances?</li>
<li>Is your location ADA accessible? (Americans with Disability Act)</li>
<li>Will you be cutting into the sidewalk or street to install utilities as part this project?</li>
</ul>

<h3 id="summary">Summary</h3>

<p>This seciton provides a summary view of all information, and directs the user to the Santa Cruz Planning Department:</p>

<blockquote>
<p>Thank you for your interest in starting a business in Santa Cruz! Please contact the Planning Department with any questions regarding permits to start the business process.</p>
</blockquote>

<p>A submit button is also provided, and presumably this leads to followup with all parties.</p>

<h2 id="definitionsindaytonzoningpdf">Definitions in Dayton Zoning PDF</h2>

<h3 id="non-conformingusage">Non-Conforming Usage</h3>

<p>Non-conforming usage can mean a parcel within a given zone may have been grandfathered into a zone despite not fullfilling a given zone&#8217;s requirements, and thus can be referred to as a <em>non-conforming</em> parcel. On page 51, there is further explanation: </p>

<blockquote>
<p>non-conforming uses, buildings, lots, and structures are subject to regulations limiting their use, restoration, reconstruction, extension, and substitution.</p>
</blockquote>

<h3 id="retail">Retail</h3>

<p>On page 83, <strong>Retail</strong> is defined as the following:</p>

<blockquote>
<p>An establishment engaged in the selling or renting of goods or merchandise to the general public for personal or household consumption, and rendering services incidental to the sale of such products. Such an establishment is open to the general public during regular business hours and has display areas that are designed and laid out to attract the general public. In determining a use to be a retail use, the proportion of display area vs. storage area and the proportion of the building facade devoted to display windows may be considered. This term does not include any adult entertainment uses. This term includes, but is not limited to, artist’s studios, dry cleaning establishments, laundromats, portrait studios, and bakeries.</p>
</blockquote>

<h2 id="conditional">Conditional</h2>

<h3 id="conditionalrelatedtodistance">Conditional Related to Distance</h3>

<p>On page 313 of Dayton&#8217;s zoning documentation, the following is stated:</p>

<blockquote>
<p>Any building or other facility, on a zoning lot, used to house or exercise livestock shall be at least 200 feet from any lot in a residential zoning district.</p>
</blockquote>

<p>This might be an opportunity to build a query based on spatial analysis&#8211; i.e., a buffer of all areas that are &gt;= 200 feet from a residential zone district.</p>
<!-- Footnotes -->

<div class="footnotes">
<hr />
<ol>

<li id="fn:1">
<p><a href="http://manytricks.com/namemangler/">NameMangler</a> OSX application used to rename files. <a href="#fnref:1" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

</ol>
</div>
