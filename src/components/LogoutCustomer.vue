<template>
  <div class="text-center">
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
  </div>

  <div class="text-h4 text-center q-mt-md q-mb-md">Logout</div>
  <div class="text-title2 text-center text-positive text-bold q-mt-sm">
    Goodbye!
  </div>
</template>
<script>
import { onMounted } from "vue";
export default {
  setup() {
    onMounted(() => {
      sessionStorage.clear();
    });
    return {};
  },
};
</script>
