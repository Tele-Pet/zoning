---
title: leaflet_test

---

{assets:css order:5}
http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.css
{/assets}

{assets:js order:5}
http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.js
{/assets}

{assets:inline_css}
div#map {height: 180px;}
{/assets}

{assets:inline_js}
$(document).ready(function() {
    var map = L.map('map').setView([51.505, -0.09], 13);
    var marker = L.marker([51.5, -0.09]).addTo(map);
    L.tileLayer('http://{s}.www.toolserver.org/tiles/bw-mapnik/{z}/{x}/{y}.png', {
        attribution: 'OpenStreetMap'
    }).addTo(map);
});
{/assets}
>>>>> Here is on top of the map
<div id="map"></div>
>>>>> And here is below the map