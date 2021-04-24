<template>
  <!-- <input v-model="query" /> -->

  <p>Selected Name</p>
  {{ selectedName }}
  <p>Selected Name</p>
  {{ selectedCordinate }}
  <p>Selected Cordinate</p>

  <vue-bootstrap-typeahead
    ref="k"
    v-on:keyup="change"
    :data="results"
    SelectName
    @hit="SelectName($event)"
  />
</template>
 
<script>
import { computed, onMounted, ref, watch } from "vue";
import VueBootstrapTypeahead from "vue-bootstrap-typeahead";

export default {
  components: {
    VueBootstrapTypeahead,
  },
  props: ["addMarkerFun"],

  setup(props) {
    const selectedName = ref("");
    const mainresult = ref([]);
    const selectedCordinate = ref({});

    const results = ref([]);
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
    console.log(k);

    const change = () => {
      console.log(k.value.inputValue);
      getNames(k.value.inputValue);
    };

    onMounted(() => {
      console.log(k);
    });

    const SelectName = (val) => {
      console.log(val);
      selectedName.value = val;
      var nameObject = mainresult.value.find((e) => e.display_name == val);
      console.log(nameObject);
      selectedCordinate.value.lat = nameObject.lat;
      selectedCordinate.value.lon = nameObject.lon;

      props.addMarkerFun(nameObject.lat, nameObject.lon);
    };

    return {
      change,
      k,
      results,
      getNames,
      selectedName,
      SelectName,
      selectedCordinate,
    };
  },
};
</script>

<style>
</style>