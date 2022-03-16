<script>

  export let district;

	import L from 'leaflet';

  const mapDefaults = {
    zoom: 7,
    center: [46.5183, -94.5541],
    trackResize: true,
    zoomControl: false,
    minZoom:7,
    maxZoom:12
  };

  const layerOptions = {
    tileSize: 512,
    zoomOffset: -1,
    attribution: '© <a href="https://www.mapbox.com/feedback/">Mapbox</a> © <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }

  const districtColors = [
    { color: '#1D8C47', fillColor: '#1D8C47', colorName: 'labelDarkGreen'},
    { color: '#0D57A0', fillColor: '#0D57A0', colorName: 'labelDarkBlue'},
    { color: '#C83D2D', fillColor: '#C83D2D', colorName: 'labelRed'},
    { color: '#FF6633', fillColor: '#FF6633', colorName: 'labelOrange'},
    { color: '#FBD341', fillColor: '#FBD341', colorName: 'labelYellow'}, 
    { color: '#0793AB', fillColor: '#0793AB', colorName: 'labelMedBlue'},
    { color: '#55CBDD', fillColor: '#55CBDD', colorName: 'labelLightGreen'},
    { color: '#1D8C47', fillColor: '#1D8C47', colorName: 'labelDarkGreen'},
    { color: '#0D57A0', fillColor: '#0D57A0', colorName: 'labelDarkBlue'},
    { color: '#C83D2D', fillColor: '#C83D2D', colorName: 'labelRed'},
    { color: '#FF6633', fillColor: '#FF6633', colorName: 'labelOrange'},
    { color: '#FBD341', fillColor: '#FBD341', colorName: 'labelYellow'}, 
    { color: '#0793AB', fillColor: '#0793AB', colorName: 'labelMedBlue'},
    { color: '#55CBDD', fillColor: '#55CBDD', colorName: 'labelLightGreen'}
  ]

  const oldDistrictStyle = {  
    stroke: false,
    color: '#404040',
    weight: 1.5,
    opacity: 0.9,
    fill: true,
    fillColor: '#55307E',
    fillOpacity: 0.4,
    clickable: false
  }

  const newDistrictStyle = {
    stroke: true,
    weight: 3,
    opacity: 0.9,
    fill: false,
    fillOpacity: 0.2,
    clickable: false
  }

  let map;
  let featureGroup = new L.featureGroup();
  let newDistricts = [];
  let currentDistrict;

  function createMap(container) {
    map = new L.Map(container, mapDefaults);
    map.addControl(new L.Control.Zoom({ position: 'topright' }));
    map.attributionControl.setPrefix(false);
    L.tileLayer('https://api.mapbox.com/styles/v1/mapbox/light-v10/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWlubnBvc3QiLCJhIjoicUlOUkpvWSJ9.djE93rNktev9eWRJVav6xA', layerOptions).addTo(map);
  }

  function resizeMap() {
	  if(map) { map.invalidateSize(); }
  }

  async function changeMap() {

    let zeroPaddedDistrict = (district.toString().length == 1) ? '0' : ''
    zeroPaddedDistrict = zeroPaddedDistrict + district;
    currentDistrict = district;

    let boundarySet = "state-senate-districts-"
    if (currentDistrict.toString().includes("A") | currentDistrict.toString().includes("B")) {
      if (zeroPaddedDistrict.length == 2) {zeroPaddedDistrict = "0" + zeroPaddedDistrict}
      boundarySet = "state-house-districts-"
    }

    let bufferSize = -0.003;
    if (currentDistrict == "67B") {bufferSize = -0.001}

    let [oldDistReq, newDistsReq] = await Promise.all([
      fetch('https://represent-minnesota.herokuapp.com/boundaries/' + boundarySet + '2012/' + zeroPaddedDistrict.toLowerCase() + '/simple_shape'),
      fetch('https://represent-minnesota.herokuapp.com/boundaries/simple_shape?buffered_intersect=' + boundarySet + '2012/' + zeroPaddedDistrict.toLowerCase() + '&sets=' + boundarySet + '2022&buffersize=' + bufferSize)
    ]);
    if (oldDistReq.ok && newDistsReq.ok) {
      let oldDist = await oldDistReq.json();
      let newDists = await newDistsReq.json();

      featureGroup.eachLayer(function(layer){
        featureGroup.removeLayer(layer)
      });
      newDistricts = [];

      for (let i = 0; i < newDists.objects.length; i++) {
        let layer = new L.geoJSON(newDists.objects[i].simple_shape);
        layer.setStyle(newDistrictStyle);
        layer.setStyle(districtColors[i]);
        layer.bindTooltip(newDists.objects[i].name.toString(), {
          permanent: true, 
          direction: 'center',
          className: districtColors[i].colorName
          }).openTooltip();
        featureGroup.addLayer(layer);
        newDistricts.push(newDists.objects[i].name);
      };

      let oldLayer = new L.geoJson(oldDist);
      oldLayer.setStyle(oldDistrictStyle);
      featureGroup.addLayer(oldLayer);

      map.addLayer(featureGroup);
      map.fitBounds(featureGroup.getBounds());

    }
  }

  $: if (district) {
    changeMap(district)
  } else {
    featureGroup.eachLayer(function(layer){
      featureGroup.removeLayer(layer)
    });
    if (map) {
      map.setView([46.5183, -94.5541],7);
    }
  }

  function serializer(list){
    let s = ""
    if (list.length == 1) {
      s = list[0]
    }
    for (let i = 0; i<list.length; i++) {
      if (i == list.length-1) {
        s += " and ";
      } else if (i != 0) {
        s += ", "
      }
      s += '<span class="' + districtColors[i].colorName + '"><strong>' + list[i] + '</strong></span>';
    }
    return s
  }

</script>

<style>
  .map {
    height: 500px;
    margin-bottom: 2em;
  }

  .old-district {
    background-color: #baa7ca;
    padding: 0 4px;
    border-radius: 5px;
  }

  :global(.leaflet-tooltip) {
    border: 0!important;
    background: none!important;
    font-size: 2em;
  }

  :global(.labelDarkGreen) {color: #1D8C47;}
  :global(.labelDarkBlue) {color: #0D57A0}
  :global(.labelRed) {color: #C83D2D}
  :global(.labelOrange) {color: #FF6633}
  :global(.labelYellow) {color: #FBD341}
  :global(.labelMedBlue) {color: #0793AB}
  :global(.labelLightGreen) {color: #55CBDD}

  @media screen and (max-width: 50em) {
    .info-box {
      max-width: 90%;
      font-size: .8em;
    }

    .map {
      max-width: 90%;
      height: 300px;
    }
  }

</style>
<link href='https://api.mapbox.com/mapbox.js/v3.3.1/mapbox.css' rel='stylesheet' />

<svelte:window on:resize={resizeMap} />

<div class="m-entry-excerpt info-box">
<p>{#if district}
    The land that was part of   
    <span class="old-district">{#if district.toString().includes("A")|| district.toString().includes("B") }House{:else}Senate{/if}&nbsp;District&nbsp;{district}</span> 
    is now part of 
    {#if district.toString().includes("A")|| district.toString().includes("B") }House{:else}Senate{/if}
    District{#if newDistricts.length > 1 }s{/if} {@html serializer(newDistricts)}.
  {:else}
  &nbsp;
  {/if}</p>
</div>

<div class="map" use:createMap></div>