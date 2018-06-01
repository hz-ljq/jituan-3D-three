<!--2D场景-LEAFLET.JS -->
<template>
<view-template appName='' :interval="60" @interval="" class="two-main">
  <div id="mapid"></div>
  <div id="dialog-to-three" v-if="dialogToReturnThree">
    <div id="question">
      Are you gonna to return to 3-Dimension scene ?
    </div>
    <div id="yes" @click="close2D('yes')">Yes</div>
    <div id="no" @click="close2D('no')">No</div>
    <div id="close" @click="close2D('close')">×</div>
  </div>
</view-template>
</template>

<script>
import './TwoMain.scss'
import 'leaflet/dist/leaflet.css'
import L from 'leaflet'

export default {
  name: '',
  props: [],
  data() {
    return {
      dialogToReturnThree: false,
      mymap: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      // setTimeout(() => {
      //   window.Bus.$emit('viewSwitch', 'to3D');
      // }, 5000);

      this.mymap = L.map('mapid').setView([30, 80], 2);
      L.tileLayer('http://mt3.google.cn/vt/lyrs=m@162000000&hl=zh-CN&gl=cn&x={x}&y={y}&z={z}', {
        // attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="http://mapbox.com">Mapbox</a>',
        minZoom: 1,
        maxZoom: 18
        // detectRetina: true
        // id: 'mapbox.streets',
        // accessToken: 'your.mapbox.access.token'
      }).addTo(this.mymap);

      this.mymap.on('zoomend', (e) => {
        if (e.target._zoom === 1) {
          this.dialogToReturnThree = true;
          this.mymap.scrollWheelZoom = false;
        }
      });
      // this.mymap.panTo([10, 10], {
      //   animate: true,
      //   duration: 2
      // })
      //
      function onMapClick(e) {
        console.log(e.latlng);
      }
      //
      this.mymap.on('click', onMapClick);
    });
  },
  methods: {
    close2D(flag) {
      this.dialogToReturnThree = false;
      if (flag === 'yes') {
        window.Bus.$emit('close2D', '');
      }
    }
  },
  beforeDestroy() {

  },
  components: {
  }
}
</script>
