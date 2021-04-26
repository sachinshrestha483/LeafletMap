<template>
  <div class="d-flex flex-row bd-highlight mb-3">
    <div class="p-2 bd-highlight">
      <SearchSection
        :addMarkerFun="addMarker"
        :initialPosition="initialPosition"
        :finalPosition="finalPosition"
        :setFinalPosMarker="setFinalPosMarker"
        :setInitialPosMarker="setInitialPosMarker"
        :createRoute="CreateRoute"
        :intermediatePoints="intermediatePoints"
        :SetIntermediatePosMarker="SetIntermediatePosMarker"
      />
    </div>
    <div class="p-2 bd-highlight">
      <div id="mapid" ref="map"></div>
    </div>
  </div>

  <!-- <div class="row">
    <div class="col-md-2">
      <SearchSection/>
    </div>

    <div class="col-md-10" >
      <div id="mapid" ref="map"></div>
    </div>
  </div> -->
</template>

<script>
import { onMounted, ref } from "vue";
// import 'node_modules/leaflet-geosearch/dist/geosearch.css';
import VueBootstrapTypeahead from "vue-bootstrap-typeahead";
import SearchSection from "@/components/SearchSection";
export default {
  components: {
    VueBootstrapTypeahead,
    SearchSection,
  },
  setup() {
    const map = ref(null);
    const initialPosition = ref({ lat: "", lon: "", name: "" });
    const finalPosition = ref({ lat: "", lon: "", name: "" });
    var prevInitalPositionPointer = null;
    var prevFinalPositionPointer = null;
    var previousRoute = null;
    const intermediatePoints = ref([]);
    var previousIntermediatePointer = [];

    var mymap;
    onMounted(() => {
      console.log(map);
      console.log("Making The map");
      mymap = L.map(map.value).setView([23.3315, 75.0367], 5);
      const tileUrl = "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";

      const attribution =
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors';

      const tileLayer = L.tileLayer(tileUrl, { attribution });
      tileLayer.addTo(mymap);
    });

    const SetIntermediatePosMarker = () => {
      if (previousIntermediatePointer.length != 0) {
        previousIntermediatePointer.forEach((item) => {
          console.log("clearing array");
          mymap.removeLayer(item);
        });
        previousIntermediatePointer = [];
      }

      intermediatePoints.value.forEach((e, index) => {
        console.log("-----------drawing------------");
        var markerOptions = {
          title: "sdsd",
          clickable: true,
          draggable: true,
        };
        let marker = L.marker([e.lat, e.lon], markerOptions);
        previousIntermediatePointer.push(marker);
        console.log("Adding Marker");

        marker.on("moveend", function (e) {
          console.log("marker drag event");
          console.log(e.target._latlng);
          // console.log("new Lat lng:"+e.latlng)
          var obj = e.target._latlng;

          intermediatePoints.value[index].lat = obj.lat;
          intermediatePoints.value[index].lon = obj.lng;
        
        });

        marker.addTo(mymap);
      });
    };

    const setInitialPosMarker = (lat, lon) => {
      if (prevInitalPositionPointer != null) {
        mymap.removeLayer(prevInitalPositionPointer);
      }
      var markerOptions = {
        title: "Initial Position marker",
        clickable: true,
        draggable: true,
      };
      console.log("Add Marker");
      var marker = L.marker([lat, lon], markerOptions);

      prevInitalPositionPointer = marker;

      marker.on("moveend", function (e) {
        console.log("marker drag event");
        console.log(e.target._latlng);
        // console.log("new Lat lng:"+e.latlng)
        var obj = e.target._latlng;

        initialPosition.value.lat = obj.lat;
        initialPosition.value.lon = obj.lng;
   
      });

      marker.addTo(mymap);
    };
    const setFinalPosMarker = (lat, lon) => {
      if (prevFinalPositionPointer != null) {
        mymap.removeLayer(prevFinalPositionPointer);
      }

      var markerOptions = {
        title: "Final Position Marker",
        clickable: true,
        draggable: true,
      };
      console.log("Add Marker");
      var marker = L.marker([lat, lon], markerOptions);
      prevFinalPositionPointer = marker;

      marker.on("moveend", function (e) {
        console.log("marker drag event");
        console.log(e.target._latlng);

        var obj = e.target._latlng;

        finalPosition.value.lat = obj.lat;
        finalPosition.value.lon = obj.lng;
      
        // console.log("new Lat lng:"+e.latlng)
      });

      marker.addTo(mymap);
    };

    const addMarker = (lat, lon) => {
      console.log(lat);
      console.log(lon);
      var markerOptions = {
        title: "MyLocation",
        clickable: true,
        draggable: true,
      };
      console.log("Add Marker");
      var marker = L.marker([lat, lon], markerOptions);
      marker.on("moveend", function (e) {
        console.log("marker drag event");
        console.log(e.target._latlng);
        // console.log("new Lat lng:"+e.latlng)
      });
      // Adding marker to the map
      marker.addTo(mymap);
    };

    const CreateRoute = (p, pathNum = 0) => {
      let possiblePaths = p;
      console.log("Creating Routes");
      console.log(p);
      console.log("Creating Routes");

      if (previousRoute != null) {
        mymap.removeLayer(previousRoute);
      }

      console.log("Path here ");
      console.log(p);

      //  var p =paths.value[0].latLongArr;
      //  let r = possiblePaths[pathNum].geoJson.map((e) => e.reverse());
      let r = possiblePaths[pathNum].geoJson;

      //      var latlngs = [...possiblePaths];

      var polyline = L.polyline(r, { color: "red" }).addTo(mymap);

      previousRoute = polyline;

      // // zoom the map to the polyline
      mymap.fitBounds(polyline.getBounds());
    };

    return {
      map,
      addMarker,
      initialPosition,
      finalPosition,
      setInitialPosMarker,
      setFinalPosMarker,
      CreateRoute,
      intermediatePoints,
      SetIntermediatePosMarker,
    };
  },
};
</script>

<style>
#mapid {
  height: 100vh;
  width: 1000px;
}

/* #mapdiv{
  width: 400px;

} */
</style>