<template>
  <v-app>
    <v-main>
      <Viewer />
    </v-main>
  </v-app>
</template>

<script>
import Viewer from './components/Viewer';
export default {
  name: 'App',
  components: {
    Viewer
  },
};
</script>
<style>
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  overflow-y: hidden !important;
}
</style>
