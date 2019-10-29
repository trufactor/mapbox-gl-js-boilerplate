<template lang="pug">
#map
</template>
<script>
export default {
  name: 'app',
  data(){
    return {
      map: null,
      options: {
        zoom: 2,
        container: 'map',
        center: [-98.5,39.8],
        style: 'mapbox://styles/mapbox/light-v9'
      }
    };
  },
  updated(){
    if(this.map) this.map.resize(); //make sure always filled screen
  },
  mounted(){
    this.map = new mapboxgl.Map(this.options);
    this.map.on('load',()=>{
      console.log('mapbox loaded');
    });
  }
};
</script>
<style lang="stylus" scoped>
#map
  height 100vh
  width 100vw
</style>
