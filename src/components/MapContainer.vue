<template>
  <div ref="map"
       style="width: 100%; height: 100%">
  </div>
  <select v-model="mode">
    <option value="modify">select a feature to modify</option>
    <option value="newPoly">draw new Poly</option>
    <option value="newPoint">draw new Point</option>
    <option value="newCircle">draw new Circle</option>
    <option value="newLine">draw new Line</option>
    <option value="newNew">draw new New</option>
  </select>
  <button @click="saveZones()">Сохранить</button>
</template>

<script>
import GeoJSON from 'ol/format/GeoJSON';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import {Draw, Modify, Select, Snap} from 'ol/interaction';
import {Map, View} from 'ol/index';
import {fromLonLat,toLonLat} from 'ol/proj';
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
      point:'',
      circle:'',
      source:'',
      modify:'',
      polygon:'',
      line:'',
      newnew:'',
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
    this.polygon=new Draw({
      type: 'Polygon',
      geometryName:'poly',
      source: this.source,
    })
    this.line=new Draw({
      type: 'LineString',
      geometryName:'line',
      source: this.source,
    })
    this.circle=new Draw({
      type: 'Circle',
      source: this.source,
    })
    this.point=new Draw({
      type: 'Point',
      source: this.source,
    })
    this.newnew=new Draw({
      type: 'MultiPoint',
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
      this.olMap.removeInteraction(this.polygon);
      this.olMap.removeInteraction(this.point);
      this.olMap.removeInteraction(this.line);
      this.olMap.removeInteraction(this.newnew)
      this.olMap.removeInteraction(this.circle);
      this.olMap.removeInteraction(this.select);
},
    onChange() {
      this.removeInteractions();
      switch (this.mode) {
        case 'newPoly': {
          this.olMap.addInteraction(this.polygon);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'modify': {
          this.olMap.addInteraction(this.select);
          this.olMap.addInteraction(this.modify);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'newPoint': {
          this.olMap.addInteraction(this.point);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'newCircle': {
          this.olMap.addInteraction(this.circle);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'newLine': {
          this.olMap.addInteraction(this.line);
          this.olMap.addInteraction(this.snap);
          break;
        }
        case 'newNew': {
          this.olMap.addInteraction(this.newnew);
          this.olMap.addInteraction(this.snap);
          break;
        }
        default: {
          // pass
        }
      }
    },
    saveZones(){
      const fet=this.source.getFeatures()
      fet.forEach((item) => {
        console.log(item)
        console.log(item.getGeometry())
        const name=item.getGeometryName()
        console.log('Name:'+name)
        const coords=item.values_[name].getCoordinates()
        console.log(coords)
        coords.forEach(function (it){
          console.log(toLonLat(it))
        })
      })
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
