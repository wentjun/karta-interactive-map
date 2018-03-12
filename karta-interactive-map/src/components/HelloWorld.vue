<template>
  <section class="container">
    <div class="left-half">
      <section id='map'>
      </section>
    </div>
    <div class="right-half">
      <h1>Welcome to Karta.</h1>
      <h3>Where would you like to go today?</h3>

      <div id='geocoder' class='geocoder'></div>
      <div v-if="resultName.length>0&&resultAddress.length>0">
        <h1>You have selected {{ resultName }}.</h1>
        <p>It is situated at {{ resultAddress }}.</p>
        <p>If you would like to get to {{ resultName }}, you may make use of the directional input on the top left hand corner of the map.</p>
      </div>
    </div>
  </section>
</template>

<script>
const config = require('../../config/app.config').default
const mapboxgl = require('mapbox-gl/dist/mapbox-gl.js')
const MapboxGeocoder = require('@mapbox/mapbox-gl-geocoder');

const defaultLatLng = {
  lat: 1.306727,
  lng: 103.788309
}

const defaultOptions = {
  container: 'map',
  style: config.mapStyles.default,
  center: [defaultLatLng.lng, defaultLatLng.lat],
  zoom: 15,
  attributionControl: false
}

mapboxgl.accessToken = config.mapboxToken

export default {
  name: 'HelloWorld',
  data () {
    return {
      route: [],
      lat: defaultLatLng.lat,
      lng: defaultLatLng.lng,
      mapType: 'default',
      resultName: '',
      resultAddress: ''
    }
  },
  mounted () {
    this.loadMap();
  },
  methods: {

    changeMapType (event) {
      this.map.setStyle(config.mapStyles[this.mapType]);
    },

    // loads map with controls
    loadMap () {
      let map;
      let vm = this;

      // initialise map
      map = new mapboxgl.Map(defaultOptions);

      // navigation control for the map (top right hand corner of the map)
      map.addControl(new mapboxgl.NavigationControl());

      // directional inputs for origin and destination
      map.addControl(new MapboxDirections({
        accessToken: mapboxgl.accessToken
      }), 'top-left');

      // search bar to search for locations
      let geocoder = new MapboxGeocoder({
        accessToken: mapboxgl.accessToken
      });
      document.getElementById('geocoder').appendChild(geocoder.onAdd(map));

      map.on('load', function() {
        map.addSource('single-point', {
          "type": "geojson",
          "data": {
            "type": "FeatureCollection",
            "features": []
          }
        });

        map.addLayer({
          "id": "point",
          "source": "single-point",
          "type": "circle",
          "paint": {
            "circle-radius": 10,
            "circle-color": "#007cbf"
          }
        });

        // Listen for the `geocoder.input` event that is triggered when a user
        // makes a selection and add a symbol that matches the result.
        geocoder.on('result', (ev) => {

          vm.resultName = ev.result.text;
          vm.resultAddress = ev.result.place_name;
          map.getSource('single-point').setData(ev.result.geometry);

          //input boxes to allow the user to search for more landmarks/transit stops. Commented it out as it is incomplete.
          /**
          document.getElementById('geocoder').appendChild(geocoder.onAdd(map));

          geocoder.on('result', (ev) => {
            console.log(ev.result.center);
            let directionsFrom = {
              "type": "FeatureCollection",
              "features": []
            }
            let point = {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": []
                }
            }
            point.geometry.coordinates = ev.result.center
            directionsFrom.features.push(point)
            console.log(directionsFrom)

            map.addSource("poi", {
                "type": "geojson",
                "data": directionsFrom
            });

            map.addLayer({
                "id": "poi-route",
                "type": "circle",
                "source": "poi",
                "paint": {
                    "circle-radius": 6,
                    "circle-color": "#B42222"
                },
                "filter": ["==", "$type", "Point"],
            });

          });
          **/

        });

      });
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

#map {
  position:absolute;
  top:0;
  bottom:0;
  width:50%;
}

#mapType {
  background: white;
  border-radius: 5px;
  box-sizing: border-box;
  display: inline;
  padding: 5px;
  top: 10px;
  right: 10px;
  position: absolute;
  z-index: 200;

  label {
    display: block;
    font-weight: bold;
  }
}

html, body, section {
  height: 100%;
}

div {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.container {
  display: flex;
}

.left-half {
  flex: 1;
  padding-right: 2vw;
}

.right-half {
  flex: 1;
  padding: 2vw;
}

.geocoder {
  width:50%;
  left:50%;
  top:20px;
  border: 1px solid grey;
  border-radius: 5px;
}

.mapboxgl-ctrl-geocoder {
  min-width:100%;
}

.mapboxgl-ctrl-directions {
  visibility: hidden !important;
}
</style>
