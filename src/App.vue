<template lang="pug">
#map
</template>
<script>
import {Trufactor} from '@trufactor/core';

export default {
  name: 'app',
  data(){
    return {
      map: null,
      db: new Trufactor({domain: 'demo'}),
      trufactorData: {
        type: 'geojson',
        data: {
          features: [],
          type: 'FeatureCollection'
        }
      },
      heatmapLayer: {
        id: 'heatmap-hex',
        type: 'heatmap',
        source: 'trufactor-data',
        paint: {

          // Color ramp for heatmap.  Domain is 0 (low) to 1 (high).
          // Begin color ramp at 0-stop with a 0-transparancy color
          // to create a blur-like effect.
          'heatmap-color': [
            'interpolate',
            ['linear'],
            ['heatmap-density'],
            0, 'rgba(33,102,172,0)',
            0.2, 'rgba(103,169,207,0.3)',
            0.4, 'rgba(209,229,240,0.35)',
            0.6, 'rgba(253,219,199,0.4)',
            0.8, 'rgba(239,138,98,0.5)',
            1, 'rgba(178,24,43,0.6)'
          ],

          // Adjust the heatmap radius by zoom level
          'heatmap-radius': 50
        }
      },
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
    this.map = new mapboxgl.Map(this.options); //initialize map

    // this lifecycle hooks is called when mapbox is loaded and ready
    this.map.on('load',()=>{
      this.map.addSource('trufactor-data',this.trufactorData);
      this.map.addLayer(this.heatmapLayer);

      // this is a lifecycle hook called after the api caches the returned
      // data on the client
      this.db.afterGetData = ({data})=>{

        // update the maps data
        this.trufactorData.data.features = data.features||[];

        // update mapbox to reflect the data change
        this.map.getSource('trufactor-data').setData(this.trufactorData.data);

        // now we need to normalize the heatmap based on the max and min totals
        const max = Math.max(...[
          ...this.trufactorData.data.features.map(f=>f.properties.total),
          0 //Math.max([]) is -Infinity, we prevent this here
        ])||1; //need max to be >0 or mapbox will complain

        // increase the heatmap weight based on frequency and property magnitude
        this.heatmapLayer.paint['heatmap-weight'] = [
          'interpolate',
          ['linear'],
          ['get', 'total'],
          0, 0,
          max/16, 0.2,
          max/8, 0.4,
          max/2, 0.6,
          max/4*3, 0.8,
          max, 1
        ];

        // Update the heatmaps weight based on the potentially updated
        // maximum value provided by the updated data
        this.map.setPaintProperty(
          'heatmap-hex',
          'heatmap-weight',
          this.heatmapLayer.paint['heatmap-weight']
        );
      };

      // go ahead and initialize the first api data call
      this.db.getData({
        zoom: this.map.getZoom()/22, //normalize level between 0 & 1
        query: this.map.getBounds()
      });
    });
  }
};
</script>
<style lang="stylus" scoped>
#map
  height 100vh
  width 100vw
</style>
