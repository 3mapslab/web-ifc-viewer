<template>
  <div class="viewer">
    <v-snackbar
      app
      tile
      darken
      top
      permanent
      v-model="selectFileVisible"
      color="primary"
      :timeout="-1"
    >
      <v-file-input v-model="ifcFile" dense accept=".ifc" label="IFC file"></v-file-input>
    </v-snackbar>
    <ThreeScene :resize="resize" :ifc="ifcFile" />
  </div>
</template>

<script>
/* eslint-disable */
import ThreeScene from '@/components/ThreeScene';
export default {
  props: ['resize'],
  components: {
    ThreeScene
  },
  data() {
    return {
      selectFileVisible: true,
      ifcFile: null,
    };
  },
  watch: {
    resize() {
      this.$nextTick(() => {
        window.dispatchEvent(new Event('resize'));
      });
    }
  },
  mounted() {
    this.initialize();
  },
  methods: {
    initialize() {}
  }
};
</script>

<style scoped>
.viewer {
  height: 100%;
  width: 100%;
}
</style>
