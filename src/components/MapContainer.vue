<template>
  <div style="display: flex;flex-direction: column;width: 70vw;height: 95vh">
    <div style="width: 100%; height: 100%">
      <div ref="map" style="width: 100%; height: 100%">
        <div class="attr">
          <button><i v-if="!newGeo.mode" @click="newGeo.mode=!newGeo.mode" class="icofont-ui-add"></i><i v-else @click="removeInteractions();newGeo.mode=!newGeo.mode" class="icofont-stop"></i></button>
          <div v-if="newGeo.mode">
            <button @click="onChange('newPoly')"><i class="icofont-polygonal"></i></button>
            <button @click="onChange('newPoint')"><i class="icofont-ui-pointer"></i></button>
            <button @click="onChange('newCircle')"><i class="icofont-circle-ruler"></i></button>
            <button @click="onChange('newLine')"><i class="icofont-line-messenger"></i></button>
            <button><i @click="modifyGeo" class="icofont-edit"></i></button>
            <input v-model="newGeo.name" minlength="6" type="text">
          </div>
        </div>
      </div>
    </div>
  </div>
  <div style="display: flex;flex-direction: column;width: 30vw;height: 95vh">
    <button @click="consoleLog(olMap)">Map console</button>
    <h3>Слои карты</h3>
    <table>
      <tbody v-if="load">
      <template v-for="layer in olMap.getAllLayers()" v-bind:key="layer">
        <tr>
          <td>
            <input @click="olMap.render()" type="checkbox" v-model="layer.values_.visible">{{ layer.values_.visible }}
          </td>
        </tr>
      </template>
      </tbody>
    </table>
    <h3>Геозоны</h3>
    <table>
      <tbody>
      <template v-if="load && !source.isEmpty()">
        <template v-for="(item,index) in source.getFeatures()" v-bind:key="item.getGeometryName()+String(index)">
          <tr>
            <td>{{ item.values_.name }}</td>
            <td>{{ item.getGeometry().getType() }}</td>
            <td>
              <button @click="source.removeFeature(item)">Удалить</button>
            </td>
          </tr>
        </template>
        <tr>
          <td>
            <button @click="modifyGeo(item)">Изменить</button>
          </td>
          <td>
            <button @click="delSelected()">Удалить выбранное</button>
          </td>
          <td><button @click="removeInteractions()">Закончить редактирование</button></td>
        </tr>
      </template>
      </tbody>
    </table>
    <div>
      <button @click="onChange('newPoly')"><i class="icofont-polygonal"></i></button>
      <button @click="onChange('newPoint')"><i class="icofont-ui-pointer"></i></button>
      <button @click="onChange('newCircle')"><i class="icofont-circle-ruler"></i></button>
      <button @click="onChange('newLine')"><i class="icofont-line-messenger"></i></button>
      <input v-model="newGeo.name" minlength="6" type="text">
    </div>
    <button @click="saveNewFeache()">Сохранить созданные зоны</button>
    <button @click="saveZones()">Сохранить Зоны в объект</button>
  </div>
</template>

<script>
import GeoJSON from 'ol/format/GeoJSON';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import {Draw, Modify, Select, Snap} from 'ol/interaction';
import {Map, View} from 'ol/index';
import {fromLonLat, toLonLat} from 'ol/proj';
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import {Collection, Feature} from "ol";
import {Circle, LineString, Point, Polygon} from 'ol/geom';
import { MousePosition, Zoom} from "ol/control";

export default {
  name: 'MapContainer',
  components: {},
  props: {},
  data() {
    return {
      olMap: '',
      myControl: new Collection([
        new Zoom({
          className:'mapZoom'
        }),
          new MousePosition({
            coordinateFormat:function(coordinate) {
              const c= toLonLat(coordinate);
              return 'Lon: '+c[0].toFixed(5)+' Lat: '+c[1].toFixed(5)
            },
            className:'mousePos'
          })
      ]),
      mode: 'modify',
      select: new Select(),
      point: '',
      circle: '',
      source: new VectorSource({
        format: new GeoJSON()
      }),
      newSource: new VectorSource({
        format: new GeoJSON()
      }),
      polygon: '',
      line: '',
      snap: '',
      load: false,
      newGeo: {
        mode: false,
        name: 'Название',
        interactions: [],

      },
      geoFances:
          [{
            "type": "Polygon",
            "name": "Тестовый полигон",
            "coords": [[37.76877651290139, 55.33576607696992], [37.62399203825214, 55.33490679709547], [37.46469028043964, 55.345840666467495], [37.22848422575214, 55.39422582045404], [36.97902412185101, 55.52357023527688], [37.01060981521039, 55.61053992998541], [36.974512764622254, 55.65862985526195], [36.89250678786664, 55.67176886555495], [36.87078109560214, 55.72005914381316], [36.943565519430265, 55.92449609210101], [37.025962980367765, 56.019021002474545], [37.31028755736935, 56.12418181060548], [37.41328438354123, 56.190719653952954], [37.569122815443684, 56.191037114132286], [37.953644299818684, 56.104592741265066], [38.19291391816968, 56.03772859595614], [38.47031870332593, 55.91555332928732], [38.529370216997805, 55.93478884983281], [38.57194223848218, 55.77136862861968], [38.51563730684156, 55.75591613596981], [38.47496909676087, 55.70850225352774], [38.36098594246399, 55.652754913320024], [38.392571635823366, 55.572090633751856], [38.39911708907325, 55.49634237976724], [38.33319912032326, 55.45587013403659], [38.34693203047951, 55.38884402740942], [38.29062709883889, 55.35372687926164], [38.16428432540139, 55.352165393681474], [38.07502040938576, 55.34591883555066], [37.98712978438576, 55.334984987757224], [37.93357143477638, 55.36075280235693], [37.85666713790139, 55.34201423634735]]
          }, {
            "type": "Point",
            "name": "Тестовая Точка",
            "coords": [37.02249057340839, 55.41880832762814]
          }, {
            "type": "Circle",
            "name": "Тестовый Круг",
            "coords": {"center": [37.10797793913105, 55.31686222357598], "radius": 13328.929861847311}
          }, {
            "type": "LineString",
            "name": "Название",
            "coords": [[38.573868370811525, 56.02686344586249], [37.837784386436525, 56.20449024974522], [37.074234581749025, 56.180038736035215], [36.65950069503028, 55.96234795508971], [36.651260948936525, 55.60248138139434], [37.57136592940528, 55.22205355834788], [38.68098507003028, 55.31750408951942], [38.74415645674903, 55.71249250026656]]
          }],
    }
  },
  mounted(){
    this.init()
    },
  methods: {
    init() {
      this.snap=new Snap({
        source: this.source,
      })
      this.point = {
        type: 'Point',
        source: this.newSource,
      }
      this.polygon = {
        type: 'Polygon',
        source: this.newSource
      }
      this.circle = {
        type: 'Circle',
        source: this.newSource,
      }
      this.line = {
        type: 'LineString',
        source: this.newSource,
      }
      this.olMap = new Map({
        target: this.$refs['map'],
        layers: [
          new TileLayer({
            source: new OSM(),
          }),
          new VectorLayer({
            source: this.source,
            style: {
              'fill-color': 'rgba(100, 255, 255, 0.5)',
              'stroke-color': '#335cff',
              'stroke-width': 2,
              'circle-radius': 7,
              'circle-fill-color': '#facc34',
            },
          }),
          new VectorLayer({
            source: this.newSource,
            style: {
              'fill-color': 'rgba(100, 255, 255, 0.5)',
              'stroke-color': '#ff3333',
              'stroke-width': 2,
              'circle-radius': 7,
              'circle-fill-color': '#facc34',
            },
          }),
        ],
        view: new View({
          zoom: 9,
          center: fromLonLat([37.55213985518653, 55.7449717807913]),
          constrainResolution: true,
          projection: 'EPSG:3857'
        }),
        controls:this.myControl
      })
      this.renderData()
      this.load = true
    },
    delSelected() {
      const ft = this.select.getFeatures().getArray()
      console.log(ft)
      console.log('length:' + ft.length)
      for (let i = 0; i < ft.length; i++) {
        console.log(ft[i])
        this.source.removeFeature(ft[i])
      }
    },
    removeInteractions() {
      for (let i = 0; i <= this.newGeo.interactions.length; i++) {
        this.olMap.removeInteraction(this.newGeo.interactions[i])
      }
      this.newGeo.interactions.splice(0, this.newGeo.interactions.length)
      console.log(this.newGeo.interactions)
    },
    consoleLog(item) {
      console.log(item.getAllLayers())
    },
    onChange(name) {
      this.removeInteractions()
      switch (name) {
        case 'newPoly': {
          this.polygon.geometryName = this.newGeo.name
          this.newGeo.interactions.push(new Draw(this.polygon))
          this.newGeo.interactions.push(this.snap)
          this.newGeo.interactions.push(new Modify({
            source: this.newSource,
          }))
          for (let i = 0; i < this.newGeo.interactions.length; i++) {
            this.olMap.addInteraction(this.newGeo.interactions[i]);
          }
          break;
        }
        case 'newPoint': {
          this.point.geometryName = this.newGeo.name
          this.newGeo.interactions.push(new Draw(this.point));
          this.newGeo.interactions.push(this.snap);
          for (let i = 0; i < this.newGeo.interactions.length; i++) {
            this.olMap.addInteraction(this.newGeo.interactions[i]);
          }
          break;
        }
        case 'newCircle': {
          this.circle.geometryName = this.newGeo.name
          this.newGeo.interactions.push(new Draw(this.circle));
          this.newGeo.interactions.push(this.snap);
          for (let i = 0; i < this.newGeo.interactions.length; i++) {
            this.olMap.addInteraction(this.newGeo.interactions[i]);
          }
          break;
        }
        case 'newLine': {
          this.line.geometryName = this.newGeo.name
          this.newGeo.interactions.push(new Draw(this.line));
          this.newGeo.interactions.push(this.snap);
          for (let i = 0; i < this.newGeo.interactions.length; i++) {
            this.olMap.addInteraction(this.newGeo.interactions[i]);
          }
          break;
        }
        default: {
          // pass
        }
      }
    },
    modifyGeo() {
      this.removeInteractions()
      this.newGeo.interactions.push(this.select);
      this.newGeo.interactions.push(new Modify({
        features: this.select.getFeatures(),
      }));
      this.newGeo.interactions.push(this.snap);
      for (let i = 0; i < this.newGeo.interactions.length; i++) {
        this.olMap.addInteraction(this.newGeo.interactions[i]);
      }
    },
    saveNewFeache() {
      let fts = this.newSource.getFeatures()
      for (let i = 0; i < fts.length; i++) {
        console.log(fts[i])
        const feathe = new Feature({
          geometry: fts[i].getGeometry(),
          name: fts[i].getGeometryName(),
        })
        this.source.addFeature(feathe)
        this.newSource.removeFeature(fts[i])
      }

    },
    renderData() {
      for (let i = 0; i < this.geoFances.length; i++) {
        if (this.geoFances[i].type === 'Polygon') {
          let coords = [[]]
          this.geoFances[i].coords.forEach(function (co) {
            coords[0].push(fromLonLat(co))
          })
          const ft = new Feature({
            geometry: new Polygon(coords),
            name: this.geoFances[i].name,
          })
          this.source.addFeature(ft)
        }
        if (this.geoFances[i].type === 'Point') {
          const coords = fromLonLat(this.geoFances[i].coords)
          const ft = new Feature({
            geometry: new Point(coords),
            name: this.geoFances[i].name,
          })
          this.source.addFeature(ft)
        }
        if (this.geoFances[i].type === 'Circle') {
          const center = fromLonLat(this.geoFances[i].coords.center)
          let c = new Circle(center)
          c.setCenter(center)
          c.setRadius(this.geoFances[i].coords.radius)
          const ft = new Feature({
            geometry: c,
            name: this.geoFances[i].name,
          })
          this.source.addFeature(ft)
        }
        if (this.geoFances[i].type === 'LineString') {
          let coords = []
          this.geoFances[i].coords.forEach(function (co) {
            coords.push(fromLonLat(co))
          })
          const ft = new Feature({
            geometry: new LineString(coords),
            name: this.geoFances[i].name,
          })
          this.source.addFeature(ft)
        }
      }
    },
    saveZones() {
      let fts = []
      this.source.forEachFeature(function (ft) {
        const type = ft.getGeometry().getType()
        if (type == 'Polygon') {
          const ftcoords = ft.getGeometry().getCoordinates()[0]
          const wgs84coords = []
          ftcoords.forEach(function (coo) {
            wgs84coords.push(toLonLat(coo))
          })
          fts.push({
            type: type,
            name: ft.values_.name,
            coords: wgs84coords
          })
        }
        if (type == 'Point') {
          const ftcoords = ft.getGeometry().getCoordinates()
          const wgs84coords = toLonLat(ftcoords)
          fts.push({
            type: type,
            name: ft.values_.name,
            coords: wgs84coords
          })
        }
        if (type == 'Circle') {
          const center = toLonLat(ft.getGeometry().getCenter())
          const radius = ft.getGeometry().getRadius()
          fts.push({
            type: type,
            name: ft.values_.name,
            coords: {
              center: center,
              radius: radius
            }
          })
        }
        if (type == 'LineString') {
          const coords = ft.getGeometry().getCoordinates()
          console.log(coords)
          const wgs84coords = []
          coords.forEach(function (coo) {
            wgs84coords.push(toLonLat(coo))
          })
          fts.push({
            type: type,
            name: ft.values_.name,
            coords: wgs84coords
          })
        }
      })
      console.log(JSON.stringify(fts))
    }
  },
}
</script>
<style>
.mousePos{
  position: absolute;
  background: none;
  height:auto ;
  width: auto;
  bottom: 0.5rem;
  left: 0.5rem;
  font-weight: bold;
  font-size: small;
}
.mapZoom{
  position: absolute;
  background: #DCDCE1;
  height:auto;
  width: auto;
  bottom: 0.5rem;
  right: 0.5rem;
}
.attr{
  position: absolute;
  display: inline-flex;
  background: #DCDCE1;
  height:auto;
  width: auto;
  top: 0.5rem;
  left: 0.5rem;
  z-index: 100;
}
</style>