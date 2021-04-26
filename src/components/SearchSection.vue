<template>
  <!-- <input v-model="query" /> -->

  <p>Initial Position</p>
  {{ initialPosition }}
  <br />
  <vue-bootstrap-typeahead
    ref="initialPSearchBox"
    v-on:keyup="InitialPchange"
    :data="results"
    @hit="SelectInitialPosition($event)"
  />
  <br />
  <vue-bootstrap-typeahead
    ref="IntermediateSearchBox"
    v-on:keyup="intermediateChange(index)"
    :data="results"
    placeholder="Add Intermediate"
    @hit="setIntermediatePosition(intermediatePoints.length, $event)"
  />

  <div v-for="(item, index) in intermediatePoints" :key="item" class="my-2">
    <div class="row">
      <div class="col-md-10">{{ item }} {{ index }}</div>
      <div class="col-md-2" v-on:click="removeIntermeditePoint(index)">
        <h1>x</h1>
      </div>
    </div>
  </div>

  <br />

  <p>Final Position</p>
  {{ finalPosition }}
  <br />
  <vue-bootstrap-typeahead
    ref="finalPSearchBox"
    v-on:keyup="FinalPchange"
    :data="results"
    @hit="SelectFinalPosition($event)"
  />

  <div class="container">
    <h4 class="text-center pt-4">Routes</h4>

    <div
      class="bg-secondary text-white p-4 my-4"
      v-for="(item, index) in resultRoutes"
      :key="item"
    >
      <div v-on:click="createRoute(resultRoutes, index)">
        <h4>Path {{ index + 1 }}</h4>
        <!-- <p>via -{{item.via}}</p> -->
        <div class="bg-secondary text-white">
          <div>
            <h4 class="text-center">Via</h4>
          </div>

          <div class="row">
            <div class="col-md-12" v-for="item1 in item.via" :key="item1">
              {{ item1 }}
            </div>
          </div>

          <br />
          <br />
          <p>Distance - {{ Math.round(item.distance / 1000) }}</p>
          <p>Duration - {{ Math.round(item.duration / 60) }}</p>
        </div>
      </div>

      <!--  {{item}} -->
    </div>
  </div>

  <!-- <button
    class="btn btn-primary mt-4 text-center"
    v-on:click="addIntermediatePoints"
  >
    Add InterMediate Points
  </button> -->

  <button class="btn btn-primary mt-4 text-center" v-on:click="calcuteRoute">
    Show Route
  </button>
</template>
 
<script>
import { computed, onMounted, ref, watch } from "vue";
import VueBootstrapTypeahead from "vue-bootstrap-typeahead";

export default {
  components: {
    VueBootstrapTypeahead,
  },
  props: [
    "addMarkerFun",
    "initialPosition",
    "finalPosition",
    "setFinalPosMarker",
    "setInitialPosMarker",
    "createRoute",
    "intermediatePoints",
    "SetIntermediatePosMarker",
    "calculateRoute",
  ],

  setup(props) {
    console.log("----------here are props-------------");
    //console.log(props.intermediatePoints)
    const mainresult = ref([]);
    const results = ref([]);
    const resultRoutes = ref([]);
    let lRoutes = [];
    
    //  const intermediatePoints = ref([]);


    const removeIntermeditePoint = async (index) => {
      props.intermediatePoints.splice(index, 1);
      props.SetIntermediatePosMarker();

      await calcuteRoute();
    };

    const addIntermediatePoints = () => {
      console.log(typeof props.intermediatePoints);
      props.intermediatePoints.push({
        name: "",
        lat: 0,
        lon: 0,
        intermediateInput: null,
      });
    };

    const intermediateChange = (index) => {
      console.log(typeof props.intermediatePoints);
      console.log("--- Here is----");
      console.log("index:");
      console.log(index);

      console.log(IntermediateSearchBox.value.inputValue);

      console.log("--- Here is----");

      getNames(IntermediateSearchBox.value.inputValue);
    };

    const setIntermediatePosition = async (index, val) => {
      console.log("Index" + index);
      console.log("Val" + val);

      console.log(props.intermediatePoints);

      var nameObject = mainresult.value.find((e) => e.display_name == val);
      console.log(nameObject);

      // props.intermediatePoints[index].lat = nameObject.lat;
      // props.intermediatePoints[index].lon = nameObject.lon;

      props.intermediatePoints.push({
        name: val,
        lat: nameObject.lat,
        lon: nameObject.lon,
      });
      console.log(props.intermediatePoints);

      props.SetIntermediatePosMarker();
      await calcuteRoute();
    };

    const calcuteRoute = async () => {
      // https://routing.openstreetmap.de/routed-bike/route/v1/driving/75.8682,22.720362;77.401989,23.258486;74.91779,23.480592?overview=false&alternatives=true&steps=true
      let res = null;

      if (props.intermediatePoints.length != 0) {
        let intermediatePointsLatLonString = "";
        props.intermediatePoints.forEach((item) => {
          intermediatePointsLatLonString =
            intermediatePointsLatLonString + item.lon + "," + item.lat + ";";
        });
        console.log("----intermediate string---");
        console.log(intermediatePointsLatLonString);
        res = await fetch(
          "https://routing.openstreetmap.de/routed-bike/route/v1/driving/" +
            props.initialPosition.lon +
            "," +
            props.initialPosition.lat +
            ";" +
            intermediatePointsLatLonString +
            props.finalPosition.lon +
            "," +
            props.finalPosition.lat +
            "?overview=false&alternatives=true&steps=true&geometries=geojson"
        );
      } else {
        res = await fetch(
          "https://routing.openstreetmap.de/routed-bike/route/v1/driving/" +
            props.initialPosition.lon +
            "," +
            props.initialPosition.lat +
            ";" +
            props.intermediatePoints +
            props.finalPosition.lon +
            "," +
            props.finalPosition.lat +
            "?overview=false&alternatives=true&steps=true&geometries=geojson"
        );
      }

      const obj = await res.json();

      let routes = obj.routes;
      console.log(routes);
      // for each route we have different path

      let paths = [];
      // {
      //     name:""
      //     duration:""
      //     distance;""
      //     via:[]
      //     latlongArr:[]
      // }

      routes.forEach((element) => {
        var pathObject = {
          name: "",
          duration: "",
          distance: "",
          via: [],
          latLongArr: [],
          intermediatePlaceNames: [],
          geoJson: [],
        };

        console.log("------Legs------");
        console.log(element.legs);
        console.log("-----Legs-------");
        console.log("----path----");
        var latLongArray = [];
        var geoJsonArray = [];

        element.legs.forEach((item2) => {
          console.log(item2);
          pathObject.via = [...pathObject.via, item2.summary];

          //         console.log(item2.steps);

          var intermediatepath = item2.steps;
          //    console.log("Intermediate Path");
          //  console.log(intermediatepath);
          //console.log("Intermediate Path");

          intermediatepath.forEach((item3) => {
            // console.log(item3.maneuver.location);
            latLongArray = [...latLongArray, item3.maneuver.location];
            geoJsonArray = [...geoJsonArray, ...item3.geometry.coordinates];

            if (item3.name != "") {
              pathObject.intermediatePlaceNames = [
                ...pathObject.intermediatePlaceNames,
                item3.name,
              ];
            }
            //console.log(latLongArray);
          });
        });

        console.log("----Lat Long Array---");
        console.log(latLongArray);

        console.log("Lat Long Array Elements");

        // latLongArray.forEach((item,index)=>{
        //     console.log(item,index);
        // })

        pathObject.latLongArr = latLongArray;
        pathObject.distance = element.distance;
        pathObject.duration = element.duration;
        pathObject.geoJson = geoJsonArray;

        console.log(pathObject);
        // changes
        // let r = possiblePaths[pathNum].geoJson.map((e) => e.reverse());

        pathObject.geoJson = pathObject.geoJson.map((e) => e.reverse());
        // changes

        paths.push(pathObject);
      });

      console.log(paths);

      console.log(paths);
      resultRoutes.value = paths;
      lRoutes = paths;

      props.createRoute(paths);
    };


   
    const getNames = async (name) => {
      const res = await fetch(
        "https://nominatim.openstreetmap.org/search?q=" + name + "&format=json"
      );
      mainresult.value = await res.json();
      console.log(mainresult.value);
      results.value = mainresult.value.map((e) => {
        return e.display_name;
      });
      console.log(results.value);
    };
    const k = ref(null);
    const initialPSearchBox = ref(null);
    const finalPSearchBox = ref(null);
    const IntermediateSearchBox = ref(null);

    console.log(k);

    const InitialPchange = () => {
      console.log("-------Change it------");
      console.log(initialPSearchBox.value.inputValue);
      console.log(initialPSearchBox.value.inputValue);
      getNames(initialPSearchBox.value.inputValue);
    };

    const FinalPchange = () => {
      console.log("-------Change it------");
      console.log(finalPSearchBox.value.inputValue);
      console.log(finalPSearchBox.value.inputValue);
      getNames(finalPSearchBox.value.inputValue);
    };

    onMounted(() => {
      console.log(k);
    });

    const SelectFinalPosition = (val) => {
      console.log(val);
      props.finalPosition.name = val;
      var nameObject = mainresult.value.find((e) => e.display_name == val);
      console.log(nameObject);

      props.finalPosition.lat = nameObject.lat;
      props.finalPosition.lon = nameObject.lon;

      props.setFinalPosMarker(props.finalPosition.lat, props.finalPosition.lon);
    };

    const SelectInitialPosition = (val) => {
      console.log(val);
      console.log(props);
      props.initialPosition.name = val;
      var nameObject = mainresult.value.find((e) => e.display_name == val);
      console.log(nameObject);
      props.initialPosition.lat = nameObject.lat;
      props.initialPosition.lon = nameObject.lon;
      props.setInitialPosMarker(
        props.initialPosition.lat,
        props.initialPosition.lon
      );
    };

    return {
      k,
      results,
      getNames,
      SelectFinalPosition,
      SelectInitialPosition,
      initialPSearchBox,
      finalPSearchBox,
      InitialPchange,
      FinalPchange,
      calcuteRoute,
      resultRoutes,
      addIntermediatePoints,
      intermediateChange,
      setIntermediatePosition,
      IntermediateSearchBox,
      removeIntermeditePoint,
      calcuteRoute
    };
  },
};
</script>

<style>
</style>