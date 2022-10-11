<template>
  <div ref="map"
       style="width: 100%; height: 100%">
  </div>
  <select v-model="mode">
    <option value="modify">select a feature to modify</option>
    <option value="draw">draw new features</option>

  </select>
  <button @click="saveZones()">Сохранить</button>
</template>

<script>
import GeoJSON from 'ol/format/GeoJSON';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import {Draw, Modify, Select, Snap} from 'ol/interaction';
import {Map, View} from 'ol/index';
import {fromLonLat} from 'ol/proj';
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
//useGeographic();

export default {
  name: 'MapContainer',
  components: {},
  props: {},
  data(){
    return{
      olMap:'',
      mode:'modify',
      select:'',
      //delete:'',
      source:'',
      modify:'',
      draw:'',
      snap:''
    }
  },
  mounted(){
    this.select=new Select()
    //this.delete=new Delete()
    this.source=new VectorSource({
      format: new GeoJSON(),
    })
    this.modify=new Modify({
      features: this.select.getFeatures(),
    })
    this.draw=new Draw({
      type: 'Polygon',
      source: this.source,
    })
    this.snap=new Snap({
      source: this.source,
    })
    this.olMap=new Map({
      target: this.$refs['map'],
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
        new VectorLayer({
          source: this.source,
        }),
      ],
      view: new View({
        zoom: 12,
        center: fromLonLat([37.55213985518653, 55.7449717807913]),
        constrainResolution: true,
        projection:'EPSG:3857'
      })
    })
    },
  methods:{
    removeInteractions() {
      this.olMap.removeInteraction(this.modify);
      this.olMap.removeInteraction(this.select);
      this.olMap.removeInteraction(this.draw);
      this.olMap.removeInteraction(this.select);
},
    onChange() {
      this.removeInteractions();
      switch (this.mode) {
        case 'draw': {
          this.olMap.addInteraction(this.draw);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'modify': {
          this.olMap.addInteraction(this.select);
          this.olMap.addInteraction(this.modify);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'delete': {
          this.olMap.addInteraction(this.select);
          this.olMap.addInteraction(this.modify);
          this.olMap.addInteraction(this.snap);
          break;
        }
        default: {
          // pass
        }
      }
    },
    saveZones(){
      console.log(this.source)
    }
  },
  watch: {
    'mode':function (newval){
      console.log(newval)
      this.onChange()
    }
  }
}
</script>
