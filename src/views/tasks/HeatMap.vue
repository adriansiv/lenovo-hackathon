<template>
  <div class="shadow p-4 mb-4">
    <div class="mb-6">
      <p class="text-3xl font-semibold mb-2 text-center">Heat Map</p>

      <p>
        Congratulations you've chosen the task of creating a heat map! Let's
        take a look at the tasks:
      </p>
      <ul class="p-4 mb-4">
        <li>Convert these plotted points into a heat map dependent on how long a device was in an area</li>
      </ul>

      <p>
        If there is any other data you need access to please let us know and we
        can work on getting it to you. You should be able to complete this task
        within this component but feel free to create child components if needed
        :)
      </p>
    </div>
  </div>

  <l-map style="height:60vh" :center="newPoint" v-model:zoom="zoom">
    <l-geo-json :geojson="geojson" :options="geojsonOptions" />
    <l-tile-layer
      url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      layer-type="base"
      name="OpenStreetMap"
    />
    <l-marker v-if="deviceLoaded" :lat-lng="newPoint"></l-marker>

    <l-circle v-for="item in deviceMessages" :key="item.publisher" :lat-lng="item.latLng" color="red" :fill="true" :radius="4500" ></l-circle>
  </l-map>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import { LMap, LGeoJson, LTileLayer, LMarker, LCircle } from "@vue-leaflet/vue-leaflet";
import { Watch } from "vue-property-decorator";
import { GeoOptions } from "../../types";

@Options({
  components: {
    LMap,
    LGeoJson,
    LTileLayer,
    LMarker,
    LCircle
  },
  computed: {
    device() {
      const d = this.$store.getters["groups/getGroup"](this.deviceId);

      if ('metadata' in d && d.metadata.type == 'Device') {
        return d
      }

      return {}
    },
    deviceThings() {
      const dC = this.$store.getters["things/getDeviceThings"](this.deviceId);

      return dC;
    },
    deviceMessages() {
      const m = this.$store.getters["messages/getMessages"];

      return m.map((device: any) => ({
        ...device,
        latLng: device.string_value.split(',')
      }));
    },
  },
})

export default class HeatMap extends Vue {
  // Properties
  private geojson = {
    type: 'FeatureCollection',
    features: {
      geometry: {
        type: "Point",
        coordinates: [-104.99404, 39.75621],
      },
    },
  };
  private geojsonOptions: GeoOptions = {};
  private zoom = 10;

  private deviceId = "01FHB33659RWM0X495B1M0TP11";
  private deviceLoaded = false;
  private newPoint = [51.5897, -0.0409197];

  // Computed Properties
  private device: any;
  private deviceThings: any;
  private deviceMessages: any;

  // Vue Lifecycle Functions
  beforeUnmount() {
    this.$store.dispatch("messages/reset");
  }

  // Methods

  // Watcher Functions
  // TODO think of cleaner way to implement this
  @Watch("device", { immediate: true })
  deviceThingsChange(newVal: any) {
    if (!("id" in newVal) || this.deviceLoaded) {
      return;
    }

    const name = "geolocation";
    const params = `device=${this.device.id}&name=${name}`;

    this.$store.dispatch("messages/getMessages", {
      params: params,
      offset: 0,
      limit: 200,
    });

    this.newPoint = [
      this.device.metadata.geolocation.latitude,
      this.device.metadata.geolocation.longitude
    ];
    this.deviceLoaded = true;
  }
}
</script>

<style lang="scss" scoped></style>
