<template>
  <div class="text-center q-mt-md">
    <q-avatar class="q-mb-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
    <div class="text-h6" style="color: blue">Find 3 Closest Branches to</div>

    <div>
      <q-input
        class="q-ma-lg text-h5"
        placeholder="enter current address"
        id="address"
        v-model="state.address"
      />
      <br />
    </div>
    <q-btn label="Find 3" @click="genMap" class="q-mb-md" style="width: 30vw" />
    <div
      style="height: 50vh; width: 90vw; margin-left: 5vw; border: solid"
      ref="mapRef"
      v-show="state.showmap === true"
    ></div>
  </div>
</template>
<script>
import { ref, reactive } from "vue";
import { fetcher } from "src/utils/apiutil";
export default {
  name: "BranchLocator",
  setup() {
    const mapRef = ref(null);
    let state = reactive({
      status: "",
      address: "",
      showmap: false,
    });
    const genMap = async () => {
      try {
        state.showmap = true;
        const tt = window.tt;
        let url = `https://api.tomtom.com/search/2/geocode/${state.address}.json?key=skwtG7Ku9T8DNpUNoPElNtT8UnJr9HCi`;
        let response = await fetch(url);
        let payload = await response.json();
        let lat = payload.results[0].position.lat;
        let lon = payload.results[0].position.lon;
        let map = tt.map({
          key: "skwtG7Ku9T8DNpUNoPElNtT8UnJr9HCi",
          container: mapRef.value,
          source: "vector/1/basic-main",
          center: [lon, lat],
          zoom: 10,
        });

        payload = await fetcher(`branch/${lat}/${lon}`);
        const threeBranches = payload;

        threeBranches.forEach((branch) => {
          let marker = new tt.Marker()
            .setLngLat([branch.longitude, branch.latitude])
            .addTo(map);
          let popupOffset = 25;
          let popup = new tt.Popup({ offset: popupOffset });
          popup.setHTML(
            `<div id="popup">Branch#: ${branch.id}</div><div>${
              branch.street
            }, ${branch.city}
            <br/>${branch.distance.toFixed(2)} km</div>`
          );
          marker.setPopup(popup);
        });
      } catch (err) {
        state.status = err.message;
      }
    };

    return {
      mapRef,
      state,
      genMap,
    };
  },
};
</script>
