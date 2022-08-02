<template>
  <div class="home">
    <YandexMap
      :coords="coords"
      :zoom="10"
      :init-without-markers="true"
      :controls="controls"
      ymap-class="yandex-map-box"
    >
      <YMapMarker :coords="coords" marker-id="123" />
    </YandexMap>
    <BottomSheet
      @change-state-position="(state) => changeBottomSheetState(state)"
      :state="bottomSheetState"
    >
      <ViewSlider
        @swipe-on-top="() => changeBottomSheetState(BS_CLOSED_STATE)"
        @swipe-on-bottom="() => changeBottomSheetState(BS_OPENED_STATE)"
      />
    </BottomSheet>
  </div>
</template>

<script>
import BottomSheet from "@/components/BottomSheet.vue";
import ViewSlider from "@/components/ViewSlider.vue";
import {
  yandexMap as YandexMap,
  ymapMarker as YMapMarker,
} from "vue-yandex-maps";
import {
  BS_OPENED_STATE,
  BS_CLOSED_STATE,
  BS_SEMICLOSED_STATE,
} from "@/components/BottomSheet.vue";

export default {
  name: "HomeView",
  components: { BottomSheet, ViewSlider, YandexMap, YMapMarker },
  data() {
    return {
      BS_OPENED_STATE: BS_OPENED_STATE,
      BS_CLOSED_STATE: BS_CLOSED_STATE,
      BS_SEMICLOSED_STATE: BS_SEMICLOSED_STATE,
      coords: [55.755865, 37.61752],
      controls: ["smallMapDefaultSet"],
      bottomSheetState: "",
    };
  },
  methods: {
    changeBottomSheetState(state) {
      this.bottomSheetState = state;
    },
    clearBottomSheetState() {
      this.bottomSheetState = "";
    },
  },
};
</script>
<style lang="scss">
.yandex-map-box {
  height: calc(100vh - 78px);
  width: 100%;
}
</style>
