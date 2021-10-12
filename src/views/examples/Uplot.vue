<template>
  <div class="shadow p-4">
    <div class="mb-6 flex items-center text-center">
      <UplotVue
        :data="uplotData"
        :options="options"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component';

@Options({
  components: {

  },
  computed: {

  }
})

export default class Uplot extends Vue {
  // Properties
  private uplotData: number[][] = [];
  private options = {
    title: "Uplot Example",
    width: 500,
    height: 300,
    series: [
      {
        label: "Date",
      },
      {
        label: "",
        points: { show: false },
        stroke: "#00437a",
        fill: "#00437a22",
      },
    ],
    scales: { x: { time: false } },
  };

  // Computed Properties

  // Watcher Functions

  // Vue Lifecycle Functions
  beforeMount() {
    this.uplotData = [
      [...new Array(1000)].map((_, i) => i),
      [...new Array(1000)].map((_, i) => i % 1000),
    ];
  }

  // Methods

}
</script>
