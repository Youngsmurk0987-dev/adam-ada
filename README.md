[index (02).html](https://github.com/user-attachments/files/26084693/index.02.html)
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<title>Startup Map App</title>
<meta name="viewport" content="width=device-width, initial-scale=1" />

<link href="https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.css" rel="stylesheet" />
<script src="https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.js"></script>

<style>
html, body { margin:0; height:100%; font-family: system-ui; }
#map { height:100%; }
.panel {
  position:absolute; bottom:0; width:100%;
  background:#fff; padding:12px;
}
input, button {
  width:100%; padding:10px; margin:5px 0;
}
</style>
</head>

<body>
<div id="map"></div>

<div class="panel">
  <input id="from" placeholder="From" />
  <input id="to" placeholder="To" />
  <button onclick="startNav()">🧭 Navigate</button>
</div>

<script>
mapboxgl.accessToken = "MAPBOX_KEY";

const map = new mapboxgl.Map({
  container: "map",
  style: "mapbox://styles/mapbox/navigation-day-v1", // live traffic
  center: [-0.1870, 5.6037],
  zoom: 13,
  pitch: 60
});

map.addControl(new mapboxgl.NavigationControl());
map.addControl(new mapboxgl.GeolocateControl({
  trackUserLocation: true
}));

// 3D BUILDINGS
map.on("load", () => {
  map.addLayer({
    id: "3d",
    source: "composite",
    "source-layer": "building",
    filter: ["==", "extrude", "true"],
    type: "fill-extrusion",
    paint: {
      "fill-extrusion-height": ["get", "height"],
      "fill-extrusion-base": ["get", "min_height"],
      "fill-extrusion-opacity": 0.6
    }
  });
});

// TURN-BY-TURN + VOICE
async function startNav() {
  const from = document.getElementById("from").value;
  const to = document.getElementById("to").value;

  const geo = q =>
    fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${q}.json?access_token=${mapboxgl.accessToken}`)
    .then(r => r.json());

  const s = (await geo(from)).features[0].center;
  const e = (await geo(to)).features[0].center;

  const route = await fetch(
    `https://api.mapbox.com/directions/v5/mapbox/driving-traffic/${s},${e}?steps=true&geometries=geojson&access_token=${mapboxgl.accessToken}`
  ).then(r => r.json());

  const steps = route.routes[0].legs[0].steps;
  steps.forEach(step => {
    speechSynthesis.speak(
      new SpeechSynthesisUtterance(step.maneuver.instruction)
    );
  });
}
</script>
</body>
</html>
