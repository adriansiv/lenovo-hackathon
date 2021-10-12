<template>
  <div class="shadow p-4">
    <div class="mb-6">
      <p class="text-3xl font-semibold mb-2 text-center">Map</p>

      <l-map style="height:60vh" :zoom="zoom" @update:zoom="zoomUpdated">
        <l-tile-layer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          layer-type="base"
          name="OpenStreetMap"
        />
        <l-circle
          v-for="circle in siteCircles"
          :key="circle.id"
          :lat-lng="circle.center"
          :radius="zoomRadius"
          :color="circle.color"
          :name="circle.name"
          :fill="true"
        >
          <l-popup>{{ circle.name }}</l-popup>
        </l-circle>
        <div v-if="showDevices">
          <l-marker
            v-for="device in deviceMarkers"
            :key="device.id"
            :lat-lng="device.latLong"
          >
            <l-popup>{{ device.name }}</l-popup>
          </l-marker>
        </div>
      </l-map>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component"
import {
  LMap,
  LTileLayer,
  LCircle,
  LPopup,
  LMarker,
} from "@vue-leaflet/vue-leaflet"

@Options({
  components: {
    LMap,
    LTileLayer,
    LCircle,
    LMarker,
    LPopup
  },
  computed: {
    sites() {
      const s = this.$store.getters["groups/getSites"];
      return s;
    },
    devices() {
      const d = this.$store.getters["groups/getDevices"];
      return d;
    },
    zoomRadius() {
      const maxZoom = 20;
      const zoomRadius = (maxZoom - this.zoom) * 9000;
      console.log("Radius:: ", zoomRadius);

      return zoomRadius;
    },
    showDevices() {
      return this.zoom >= 5;
    },
    siteCircles() {
      const sitesMapped = Object.keys(this.sites).map((siteKey) => {
        const siteObj = this.sites[siteKey]
        return {
          id: siteKey,
          name: siteObj.name,
          center: [
            siteObj.metadata.geolocation.latitude,
            siteObj.metadata.geolocation.longitude,
          ],
          color: "#553c9a",
        }
      });

      return sitesMapped;
    },
    deviceMarkers() {
      const devicesMapped = Object.keys(this.devices).map((deviceKey) => {
        const device = this.devices[deviceKey]
        return {
          id: deviceKey,
          name: device.name,
          latLong: [
            device.metadata.geolocation.latitude,
            device.metadata.geolocation.longitude,
          ],
        }
      });

      return devicesMapped;
    }
  }
})

export default class Map extends Vue {
  // Properties
  private zoom = 2;

  // Computed Properties
  private sites: any;
  private devices: any;
  private zoomRadius: any;
  private showDevices: any;
  private siteCircles: any;
  private deviceMarkers: any;

  // Watcher Functions

  // Vue Lifecycle Functions

  // Methods
  private redirectDeviceOverview (id: string) {
    this.$router.push(`/dashboard/deviceoverview/${id}`);
  }

  private zoomUpdated(zoom: number) {
    console.log("New Zoom:: ", zoom);
    this.zoom = zoom;
  }
}
</script>

<style lang="scss" scoped>

</style>
