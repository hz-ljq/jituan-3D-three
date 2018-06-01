<!-- -->
<template>
<view-template appName='' :interval="60" @interval="" class="selection">
  <div class="title">传输资源3D呈现</div>

  <div class="labels">
    <div class="cable-label" @click="labelFlag=true" :class="labelFlag?'selected':'unselected'" :style="{'left': labelFlag?'240px':'180px'}">
      国际海陆缆展示
    </div>
    <div class="satellite-label" @click="labelFlag=false" :class="!labelFlag?'selected':'unselected'" :style="{'left': !labelFlag?'240px':'180px'}">
      卫星网展示
    </div>
  </div>

  <transition name="switch">
    <div class="cable-selection-area" v-show="labelFlag">
      <div class="sea-cable-selection">
        <div class="header">国际海缆</div>
        <div class="body">
          <div class="item" v-for="(item, index) in seaCables" @click="clickSeaCable(index)">
            <div class="radio">
              <div class="border"></div>
              <div class="center" :style="{'background-color':item.flag?'#36ff76':'#274469'}"></div>
            </div>
            <div class="cable-name">
              {{item.name}}
              <div class="tips" v-if="item.tips">{{item.tips}}</div>
            </div>
          </div>
        </div>
      </div>
      <div class="land-cable-selection">
        <div class="header">国际陆缆</div>
        <div class="body">
          <div class="item" v-for="(item, index) in landCables" @click="clickLandCable(index)">
            <div class="radio">
              <div class="border"></div>
              <div class="center" :style="{'background-color':item.flag?'#36ff76':'#274469'}"></div>
            </div>
            <div class="cable-name">{{item.name}}</div>
          </div>
        </div>
      </div>
    </div>
  </transition>

  <transition name="switch">
    <div class="satellite-selection-area" v-show="!labelFlag">
      <div class="header">卫星网</div>
      <div class="body">
        <div class="item" v-for="(item, index) in satelliteTypes" @click="clickSatelliteType(index)">
          <div class="radio">
            <div class="border"></div>
            <div class="center" :style="{'background-color':item.flag?'#36ff76':'#274469'}"></div>
          </div>
          <div class="type">{{item.type}}</div>

          <div class="sub-item" v-for="(subItem, subIndex) in item.satellites" @click.stop="clickSatellite(index, subIndex)">
            <div class="radio">
              <div class="border"></div>
              <div class="center" :style="{'background-color':subItem.flag?'#36ff76':'#274469'}"></div>
            </div>
            <div class="satellite-name">{{subItem.name}}</div>
          </div>

        </div>
      </div>
    </div>
  </transition>

  <!-- <div class="ring"></div> -->
  <div class="legend"></div>

  <transition name="bounce">
    <!-- <two v-if="twoFlag"></two> -->
  </transition>
</view-template>
</template>

<script>
import './SelectionMain.scss'

export default {
  name: '',
  props: [],
  data() {
    return {
      labelFlag: true,

      seaCables: [{
        flag: false,
        name: '夏金海缆',
        code: '_xiajin'
      }, {
        flag: false,
        name: '福淡海缆',
        code: '_fudan'
      }, {
        flag: false,
        name: '亚太一号(SJC)海缆',
        code: '_sjc'
      }, {
        flag: false,
        name: '亚太二号(APG)海缆',
        code: '_apg'
      }, {
        flag: false,
        name: 'FASTER海缆',
        code: '_faster'
      }, {
        flag: false,
        name: 'SMW5海缆',
        code: '_smw5'
      }, {
        flag: false,
        name: '租用海缆系统',
        code: '_zuyonghailan',
        tips: '(包括：JUS海缆、EAC海缆、TPE海缆、EIG海缆、Unity海缆、APCN2海缆、SMW4海缆)'
      }],

      landCables: [{
        flag: false,
        name: 'ITMC系统',
        code: '_itmc'
      }, {
        flag: false,
        name: '中哈俄系统-欧洲',
        code: '_zhonghae'
      }, {
        flag: false,
        name: '中蒙系统-欧洲',
        code: '_zhongmeng'
      }, {
        flag: false,
        name: '中俄系统-欧洲',
        code: '_zhonge'
      }, {
        flag: false,
        name: '中越系统',
        code: 'xxxxxx'
      }, {
        flag: false,
        name: '珠澳系统',
        code: 'xxxxxx'
      }, {
        flag: false,
        name: '粤港系统',
        code: 'xxxxxx'
      }],

      satelliteTypes: [{
        flag: false,
        type: '卫星',
        code: '_satellite',
        satellites: [{
          flag: false,
          name: '中星12号卫星',
          code: '_12'
        }, {
          flag: false,
          name: '中星6B卫星',
          code: '_6b'
        }, {
          flag: false,
          name: '中星6A卫星',
          code: '_6a'
        }]
      }, {
        flag: false,
        type: '卫星轨道',
        code: '_satelliteOrbit'
      }, {
        flag: false,
        type: '卫星覆盖范围',
        code: '_satelliteCoverRegion',
        satellites: [{
          flag: false,
          name: '中星12号卫星',
          code: '_12'
        }, {
          flag: false,
          name: '中星6B卫星',
          code: '_6b'
        }, {
          flag: false,
          name: '中星6A卫星',
          code: '_6a'
        }]
      }, {
        flag: false,
        type: '卫星站',
        code: '_satelliteStation',
        satellites: [{
          flag: false,
          name: '中星12号卫星',
          code: '_12'
        }, {
          flag: false,
          name: '中星6B卫星',
          code: '_6b'
        }]
      }]
    }
  },
  mounted() {
    this.$nextTick(() => {
      // window.Bus.$on('close2D', (params) => {
      //
      // });
    });
  },
  methods: {
    // 点击海缆目录
    clickSeaCable(index) {
      this.seaCables[index].flag = !this.seaCables[index].flag;
      // let event = this.seaCables[index].flag ? 'addSeaCables' : 'removeSeaCables';
      window.Bus.$emit('toggleCables', this.seaCables[index]);
    },

    // 点击陆缆目录
    clickLandCable(index) {
      this.landCables[index].flag = !this.landCables[index].flag;
      window.Bus.$emit('toggleCables', this.landCables[index]);

      // this.landCables[index].flag = !this.landCables[index].flag;
      // let event = this.landCables[index].flag ? 'addLandCables' : 'removeLandCables';
      // window.Bus.$emit(event, this.landCables[index].code);
      // console.log(event, this.landCables[index].code);
    },

    // 点击卫星网一级目录
    clickSatelliteType(index) {
      let type = this.satelliteTypes[index].code;

      if (!this.satelliteTypes[index].flag) {
        this.satelliteTypes[index].flag = true;
        if (type === '_satelliteOrbit') { // 卫星轨道的情况
          window.Bus.$emit('addSatellite', [type]);
        } else if (type === '_satellite' || type === '_satelliteCoverRegion') {
          this.satelliteTypes[index].satellites.map((x, subIndex) => {
            this.satelliteTypes[index].satellites[subIndex].flag = true;
            let setCornFlag = this.satelliteTypes[0].satellites[subIndex].flag && this.satelliteTypes[2].satellites[subIndex].flag;  // 该卫星与该卫星的覆盖范围是否同时选择
            window.Bus.$emit('addSatellite', [type, x.code, setCornFlag]);
          });
        } else {
          this.satelliteTypes[index].satellites.map((x, subIndex) => {
            this.satelliteTypes[index].satellites[subIndex].flag = true;
            window.Bus.$emit('addSatellite', [type, x.code]);
          });
        }
      } else {
        if (type === '_satelliteOrbit') { // 卫星轨道的情况
          this.satelliteTypes[index].flag = false;
          window.Bus.$emit('removeSatellite', [type]);
          // window.Bus.$emit('removeSatelliteOrbit');
        } else {
          let tmp = this.satelliteTypes[index].satellites.every((x) => {
            return x.flag;
          });
          this.satelliteTypes[index].flag = !tmp;
          let event = this.satelliteTypes[index].flag ? 'addSatellite' : 'removeSatellite';
          this.satelliteTypes[index].satellites.map((x, subIndex) => {
            if (x.flag !== this.satelliteTypes[index].flag) {
              this.satelliteTypes[index].satellites[subIndex].flag = this.satelliteTypes[index].flag;
              if (type === '_satellite' || type === '_satelliteCoverRegion') {
                let setCornFlag = this.satelliteTypes[0].satellites[subIndex].flag && this.satelliteTypes[2].satellites[subIndex].flag;  // 该卫星与该卫星的覆盖范围是否同时选择
                window.Bus.$emit(event, [type, x.code, setCornFlag]);
              } else {
                window.Bus.$emit(event, [type, x.code]);
              }
            }
          });
        }
      }
    },

    // 点击卫星网二级目录
    clickSatellite(index, subIndex) {
      this.satelliteTypes[index].satellites[subIndex].flag = !this.satelliteTypes[index].satellites[subIndex].flag;
      this.satelliteTypes[index].flag = false;
      for (let x of this.satelliteTypes[index].satellites) {
        if (x.flag) {
          this.satelliteTypes[index].flag = true;
          break;
        }
      }
      let event = this.satelliteTypes[index].satellites[subIndex].flag ? 'addSatellite' : 'removeSatellite';
      let type = this.satelliteTypes[index].code;
      let code = this.satelliteTypes[index].satellites[subIndex].code;
      if (type === '_satellite' || type === '_satelliteCoverRegion') {
        let setCornFlag = this.satelliteTypes[0].satellites[subIndex].flag && this.satelliteTypes[2].satellites[subIndex].flag;  // 该卫星与该卫星的覆盖范围是否同时选择
        window.Bus.$emit(event, [type, code, setCornFlag]);
      } else {
        window.Bus.$emit(event, [type, code]);
      }
      console.log(event, [type, code]);
    }
  },
  beforeDestroy() {},
  components: {}
}
</script>
