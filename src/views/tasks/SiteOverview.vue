<template>
  <div class="shadow p-4 mb-4">
    <div class="mb-6">
      <p class="text-3xl font-semibold mb-2 text-center">Site Overview Task</p>

      <p>
        Congratulations you've chosen the task of creating a site overview! Let's
        take a look at the tasks:
      </p>
      <ul class="p-4 mb-4">
        <li>Create a tree strucuture displaying a breakdown of the site</li>
        <li>The site structure is site, devices, and then components</li>
      </ul>

      <p>
        If there is any other data you need access to please let us know and we
        can work on getting it to you. You should be able to complete this task
        within this component but feel free to create child components if needed
        :)
      </p>
    </div>
  </div>

  <div class="shadow p-4">
    <p class="text-3xl font-semibold mb-2 text-center">Site Overview</p>

    <div class="flex flex-wrap -mx-3 mb-20 mb-6">
      <div class="w-1/2 xl:w-1/2 px-3">
        <div class="bg-white border text-blue-400 rounded-lg flex items-center p-6 mb-6">
          <img class="icon" src="@/assets/home.svg" />
          <div class="text-gray-700">
            <p class="font-semibold text-3xl">{{ site.name }}</p>
            <p>Site Name</p>
          </div>
        </div>
      </div>

      <div class="w-1/2 xl:w-1/2 px-3">
        <div class="w-full bg-white border text-blue-400 rounded-lg flex items-center p-6 mb-6">
          <img class="icon" src="@/assets/chip.svg" />
          <div class="text-gray-700">
            <p class="font-semibold text-3xl">{{ Object.keys(siteDevices).length }}</p>
            <p>Site Devices</p>
          </div>
        </div>
      </div>

      <div v-for="(aggregatedMessage, key) in aggregatedMessages"
           :key="key"
           class="w-1/2 xl:w-1/2 px-3"
      >
        <div class="w-full bg-white border text-blue-400 rounded-lg flex items-center p-6 mb-6">
          <img class="icon" src="@/assets/people.svg" v-if="key === 'clients'" />
          <img class="icon" src="@/assets/upload.svg" v-if="key === 'rxGigabytes'" />
          <img class="icon" src="@/assets/download.svg" v-if="key === 'txGigabytes'" />
          <div class="text-gray-700">
            <p class="font-semibold text-3xl">{{ aggregatedMessage }}</p>
            <p v-if="key === 'rxGigabytes'">Upload</p>
            <p v-else-if="key === 'txGigabytes'">Download</p>
            <p v-else>{{ key }}</p>
          </div>
        </div>
      </div>

      <div class="w-1/2">
        <div class="bg-white border text-blue-400 rounded-lg flex items-center p-6 mb-6">
          <img class="icon" src="@/assets/globe.svg" />

          <div v-if="'metadata' in site">
            <div class="w-full text-gray-700">
              <p class="font-semibold text-3xl">{{ site.metadata.geolocation.altitude }}</p>
              <p>altitude</p>
            </div>

            <div class="text-gray-700">
              <p class="font-semibold text-3xl">{{ site.metadata.geolocation.latitude }}</p>
              <p>latitude</p>
            </div>

            <div class="text-gray-700">
              <p class="font-semibold text-3xl">{{ site.metadata.geolocation.longitude }}</p>
              <p>longitude</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component'
import { Watch } from 'vue-property-decorator'

interface AggregratedMessageProps {
  rxGigabytes: number
  txGigabytes: number
  clients: number
}

@Options({
  components: {},
  computed: {
    site() {
      const s = this.$store.getters['groups/getGroup'](this.$route.params.siteId);
      if ('metadata' in s && s.metadata.type == 'Site') {
        return s;
      }
      return {};
    },
    siteDevices() {
      const sD = this.$store.getters['groups/getSiteDevices'](this.$route.params.siteId);
      return sD
    },
    siteThings() {
      const sT = this.$store.getters['things/getSiteThings'](this.$route.params.siteId);
      return sT;
    },
    siteMessages() {
      const m = this.$store.getters['messages/getMessages'];
      return m;
    },
    aggregatedMessages() {
      const deviceMessages: any = {};
      const aggregatedMessages: AggregratedMessageProps = {
        rxGigabytes: 0,
        txGigabytes: 0,
        clients: 0,
      };

      this.siteMessages.map((siteMessage: any) => {
        if (!(siteMessage.device in deviceMessages)) {
          deviceMessages[siteMessage.device] = {};
        }
        if (!(siteMessage.name in deviceMessages[siteMessage.device])) {
          if ('value' in siteMessage) {
            deviceMessages[siteMessage.device][siteMessage.name] = siteMessage.value;
          }
          if ('string_value' in siteMessage) {
            deviceMessages[siteMessage.device][siteMessage.name] = siteMessage.string_value;
          }
        }
      })

      for (const key in deviceMessages) {
        if ('rxBytes' in deviceMessages[key]) {
          aggregatedMessages.rxGigabytes += deviceMessages[key].rxBytes;
        }
        if ('txBytes' in deviceMessages[key]) {
          aggregatedMessages.txGigabytes += deviceMessages[key].txBytes;
        }
        if ('numSta' in deviceMessages[key]) {
          aggregatedMessages.clients += deviceMessages[key].numSta;
        }
      }

      aggregatedMessages.rxGigabytes /= 1000000000;
      aggregatedMessages.txGigabytes /= 1000000000;
      aggregatedMessages.rxGigabytes = Number(aggregatedMessages.rxGigabytes.toFixed(2));
      aggregatedMessages.txGigabytes = Number(aggregatedMessages.txGigabytes.toFixed(2));
      return aggregatedMessages;
    },
  },
})
export default class SiteOverview extends Vue {
  // Get last metric for each name
  // Properties
  private messageMetrics = [
    'guestNumSta',
    'guestRxBytes',
    'guestRxDropped',
    'guestRxErrors',
    'guestRxPackets',
    'guestTxBytes',
    'guestTxDropped',
    'guestTxErrors',
    'guestTxPackets',
    'guestTxRetries',
    'numSta',
    'rxBytes',
    'txBytes',
    'userNumSta',
    'userRxBytes',
    'userRxDropped',
    'userRxErrors',
    'userRxPackets',
    'userTxBytes',
    'userTxDropped',
    'userTxErrors',
    'userTxPackets',
    'userTxRetries',
  ];
  private siteLoaded = false;

  // Computed Properties
  private site: any;
  private siteDevices: any;
  private siteThings: any;
  private siteMessages: any;

  // Watcher Functions
  // TODO think of cleaner way to implement this
  @Watch('site', { immediate: true })
  siteThingsChange(newVal: any) {
    if (!('id' in newVal) || this.siteLoaded) {
      return;
    }

    const params = `site=${this.site.id}`
    this.$store.dispatch('messages/getMessages', {
      params: params,
      offset: 0,
      limit: 200,
    });
    this.siteLoaded = true;
  }

  // Vue Lifecycle Functions
  beforeUnmount() {
    this.$store.dispatch('messages/reset');
  }

  // Methods
}
</script>

<style>
.icon {
  height: 50px;
  width: 50px;
  margin-right: 10px;
}
</style>
