<!-- MarkdownTOC -->

<ul>
<li><a href="#introduction">Introduction</a></li>
<li><a href="#gisstepsnotes">GIS Steps &amp; Notes</a>

<ul>
<li><a href="#determiningprojection">Determining Projection</a></li>
<li><a href="#uniquevaluesinnew_zonecolumn">Unique values in <code>NEW_ZONE</code> column</a></li>
<li><a href="#addingfullnamestoattributecolumninqgis">Adding full names to attribute column in QGIS</a></li>
<li><a href="#deleteunneededcolumns">Delete unneeded columns</a></li>
<li><a href="#exportseparatelayersasgeojson">Export separate layers as GeoJSON</a></li>
</ul></li>
<li><a href="#understandingzoningdata">Understanding Zoning Data</a>

<ul>
<li><a href="#decodingvalues">Decoding values</a></li>
<li><a href="#codesforuseregulations">Codes for Use Regulations</a></li>
</ul></li>
</ul>
<!-- /MarkdownTOC -->

<h1 id="introduction">Introduction</h1>

<p>The goal of this project is to prepare and visualize a zoning shapefile in order to encourage and streamline local business and community initiatives.</p>

<p>Currently, each zone is represented by an individual <code>.GeoJSON</code> layer that can be viewed individually by clicking on any of the layers <a href="https://github.com/Tele-Pet/zoning/tree/master/GeoJSON_lyrs">here</a>, or all layers at once <a href="https://github.com/Tele-Pet/zoning/blob/master/GeoJSON_lyrs/zoning_4326.geojson">here</a>.</p>

<p>A rough draft of a custom Leaflet-based map has been started <a href="https://github.com/Tele-Pet/zoning/tree/master/qgis_to_leaflet/export_2015_09_06_10_35_09">here</a>. It can be downloaded and viewed locally in browser by opening the <code>index.html</code> file. Also, a working draft of the map is being hosted <a href="http://chizu.kontele.com/leaflet_test">here</a>.</p>

<p>Existing documentation detailing zoning data is available <a href="http://www.cityofdayton.org/departments/pcd/planning/Pages/ZoningMap.aspx">here</a>, and will need to be researched in order to further refine any data visualizations.</p>

<h1 id="gisstepsnotes">GIS Steps &amp; Notes</h1>

<h2 id="determiningprojection">Determining Projection</h2>

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

<h2 id="uniquevaluesinnew_zonecolumn">Unique values in <code>NEW_ZONE</code> column</h2>

<p>Presuming environment is <strong>set up to run QGIS modules externally</strong>, <a href="google.com">getUniqueAttributes_v1.py</a> can be run to print out total number of unique values in <code>NEW_ZONE</code> column, and a list of all unique names. Here are the <strong>26</strong> unique values found in <a href="google.com">zoning_aug62015.shp</a> file: </p>
<pre><code class="(null)">[NULL, u&apos;AP&apos;, u&apos;BP&apos;, u&apos;CBD&apos;, u&apos;CI&apos;, u&apos;EGC&apos;, u&apos;EMF&apos;, u&apos;ENC&apos;, u&apos;ER-3&apos;, u&apos;ER-4&apos;, u&apos;I-1&apos;, u&apos;I-2&apos;, u&apos;MGC&apos;, u&apos;MH&apos;, u&apos;MMF&apos;, u&apos;MNC&apos;, u&apos;MR-5&apos;, u&apos;OS&apos;, u&apos;SGC&apos;, u&apos;SMF&apos;, u&apos;SNC&apos;, u&apos;SR-1&apos;, u&apos;SR-2&apos;, u&apos;T&apos;, u&apos;UBD&apos;, u&apos;WO&apos;]</code></pre>

<h2 id="addingfullnamestoattributecolumninqgis">Adding full names to attribute column in QGIS</h2>

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

<h2 id="deleteunneededcolumns">Delete unneeded columns</h2>

<p>Toggle <em>on</em> edit mode and delete all columns except <code>zone_name</code>. Then, choose <em>Save as</em> option, specifying <code>.GeoJSON</code> format and <code>EPSG:4326</code> CRS.</p>

<h2 id="exportseparatelayersasgeojson">Export separate layers as GeoJSON</h2>

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

<h1 id="understandingzoningdata">Understanding Zoning Data</h1>

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
	<td style="text-align:left;">Just one instance, appears to be Westwood Elementary, Westwood Park, and some land north of the park</td>
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
<!-- Footnotes -->

<div class="footnotes">
<hr />
<ol>

<li id="fn:1">
<p><a href="http://manytricks.com/namemangler/">NameMangler</a> OSX application used to rename files. <a href="#fnref:1" title="return to article" class="reversefootnote">&#160;&#8617;</a></p>
</li>

</ol>
</div>
