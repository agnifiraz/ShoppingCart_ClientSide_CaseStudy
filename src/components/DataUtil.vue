<template>
  <div class="text-center">
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
  </div>
  <div class="text-center q-mt-lg">
    <div class="text-h4">Data Utility</div>
    <q-btn
      class="q-ma-sm"
      color="white"
      text-color="black"
      label="Load Brands"
      @click="loadBrands"
    />
    <q-btn
      class="q-ma-sm"
      color="white"
      text-color="black"
      label="Load Branches"
      @click="loadBranches"
    />
    <div class="status q-mt-md text-subtitle2 text-negative" text-color="red">
      {{ state.status }}
    </div>
  </div>
</template>
<script>
import { reactive } from "vue";
import { fetcher } from "../utils/apiutil";
export default {
  setup() {
    let state = reactive({
      status: "",
    });
    const loadBrands = async () => {
      try {
        state.status = "resetting tables ...";
        let payload = await fetcher("Data");
        state.status = payload;
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };
    const loadBranches = async () => {
      try {
        state.status = "loading store data ...";
        let payload = await fetcher("Data/loadbranches"); //wont work
        state.status = payload;
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };
    return {
      loadBrands,
      loadBranches,
      state,
    };
  },
};
</script>

<!-- <template>
  <div class="text-center q-mt-lg">
    <q-avatar class="q-mb-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
    <div class="text-h4">Data Utility</div>
    <q-btn
      class="q-ma-sm"
      color="white"
      text-color="black"
      label="Load Brands"
      @click="loadBrands"
    />
    <div class="status q-mt-md text-subtitle2 text-negative" text-color="red">
      {{ state.status }}
    </div>
  </div>
</template>
<script>
import { reactive } from "vue";
export default {
  setup() {
    let state = reactive({
      status: "",
    });
    const loadBrands = async () => {
      // replace ###### with the port number your ASP.Net core server is using
      let url = "https://localhost:7216/api/Data";
      try {
        state.status = "resetting casestudy tables ...";
        let response = await fetch(`${url}`);
        state.status = await response.json();
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };
    return {
      loadBrands,
      state,
    };
  },
};
</script> -->
