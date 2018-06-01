<!--3D场景-THREE.JS -->
<template>
  <view-template appName='' :interval="60" @interval="" class="three-main">
    <div id="canvas-frame"></div>
    <div id="distance" v-text="'Distance: '+distance"></div>
    <div id="dialog-to-two" v-if="dialogToOpenTwo">
      <div id="question">
        Are you gonna to open 2-Dimension scene ?
      </div>
      <div id="yes" @click="open2D('yes')">Yes</div>
      <div id="no" @click="open2D('no')">No</div>
      <div id="close" @click="open2D('close')">×</div>
    </div>

    <transition name="bounce">
      <!-- <component :is="currentView"></component> -->
      <two v-if="twoFlag"></two>
    </transition>
  </view-template>
</template>

<script>
import './ThreeMain.scss'
// import 'three'
import Stats from '../../../../../../static/three/Stats.js'
import '../../../../../../static/three/TrackballControls.js'
import '../../../../../../static/three/QuickHull.js'
import '../../../../../../static/three/ConvexGeometry.js'
// import '../../../../../../static/three/OrbitControls.js'
// import '../../../../../../static/three/Tween.js'
import Two from './2d/TwoMain.vue'
import chinaMap from './china'
import stationInfo from './stationsInfo'  // 基站打点的本地数据

export default {
  name: '',
  props: [],
  data() {
    return {
      urlPrefix: 'http://localhost:3000',
      curve: null,
      loader: null,
      twoFlag: false,

      pipeOnEarthMesh: null,
      pipeBetweenSatellites: null,
      nEnd: 0,
      nMax: 0,
      nStep: 18,
      nEnd2: 0,
      nMax2: 0,
      nStep2: 90,

      distance: 0,
      dialogToOpenTwo: false,
      renderTimer: 0,
      stats: null,
      width: 0,
      height: 0,
      renderer: null,
      camera: null,
      scene: null,

      // position: null,

      clock: null,
      trackballControls: null,
      // orbitControls: null,

      light2: null,
      spotLight_12: null,
      spotLight_6a: null,
      spotLight_6b: null,
      lightHelper_12: null,
      lightHelper_6a: null,
      lightHelper_6b: null,
      corn_12: null,
      corn_6a: null,
      corn_6b: null,
      // sunLight: null,
      // sunHelper: null,

      satelliteMesh_12: null,
      satelliteMesh_6a: null,
      satelliteMesh_6b: null,
      satelliteFlag_12: false,
      satelliteFlag_6a: false,
      satelliteFlag_6b: false,
      scatterMesh: null,

      earthMesh: null,
      earthFlag: false,
      moonMesh: null,
      moonFlag: false,
      spaceMesh: null,
      spaceFlag: false,
      cables: [],
      baseStationArr: [],
      earthSpin: 0,

      layersMap: {

        // 海缆
        _xiajin: null,
        _fudan: null,
        _sjc: null,
        _apg: null,
        _faster: null,
        _smw5: null,
        _zuyonghailan: null,

        // 路缆
        _itmc: null,
        _zhonghae: null,
        _zhongmeng: null,
        _zhonge: null,
        // _zhongyue: null,
        // _zhuao: null,
        // _yuegang: null

        // 卫星覆盖范围
        _6a: null,
        _6b: null,
        _12_c: null,
        _12_ku: null
      },
      satelliteOrbit: null,

      idrStationMesh_12: null,
      idrStationMesh_6b: null,
      superStationMesh_12: null,
      superStationMesh_6b: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initRenderer();
      this.initCamera();
      this.initScene();
      this.initLight();
      this.initObject();

      this.renderer.clear();
      this.renderRecursion();
      // this.initTweenMotion(); // TWEEN：设置动画行为；
    });
  },
  methods: {
    // // TWEEN：设置动画行为
    // initTweenMotion() {
    //   /* 【begin】设置TWEEN动画的初始状态 */
    //   this.position = {
    //     arc: 0
    //   }
    //
    //   let tween1 = new TWEEN.Tween(this.position)
    //     .to({
    //       arc: Math.PI * 2
    //     }, 10000)
    //     .easing(TWEEN.Easing.Quadratic.InOut) // 设置补间方式
    //     .delay(0) // 延迟执行
    //   .onUpdate(update); // 设置变量发生变化时的回调
    //
    //   // tween1.chain(tween2);
    //   // tween2.chain(tween3);
    //   // tween3.chain(tween4);
    //   // tween4.chain(tween5);
    //   // tween5.chain(tweenBack);
    //   // tweenBack.chain(tween1); // 收尾相连，无限循环
    //
    //   tween1.start(); // 执行第一步动画
    // },

    // 创建地球earth；
    addEarth() {
      return new Promise((resolve, reject) => {
        // this.loader.load('http://localhost:3000/img/earth-4k.png', (texture) => {
        this.loader.load(`/myapi/img/earth-china.jpg`, (texture) => {
          texture.anisotropy = this.renderer.getMaxAnisotropy(); // 设置纹理的各向异性为GPU的最大有效值；
          console.time('kk');
          let geo = new THREE.SphereBufferGeometry(200, 30, 30);
          let mat = new THREE.MeshPhongMaterial({
            map: texture,
            shininess: 4
            // emissive: 0xff0000
          });
          this.earthMesh = new THREE.Mesh(geo, mat);
          this.earthMesh.position.set(0, 0, 0);
          this.scene.add(this.earthMesh);
          this.earthFlag = true;
          console.timeEnd('kk');
          resolve('○○○○○○○○○○创建地球earth!!');
        })
      });
    },

    // 创建聚光灯；
    createSpotlight(color) {
      let newObj = new THREE.SpotLight(color, 2);
      newObj.castShadow = true;
      newObj.intensity = 15;
      newObj.angle = 0.04;
      newObj.penumbra = 0.8;
      newObj.decay = 1;
      newObj.distance = 500;
      newObj.shadow.mapSize.width = 1024;
      newObj.shadow.mapSize.height = 1024;
      return newObj;
    },

    // 添加图层（包括海陆缆线、卫星覆盖范围）
    addLayer(param) {
      return new Promise((resolve, reject) => {
        this.loader.load(`/myapi/img/layers/${param}.png`, (texture) => {
          let geo = new THREE.SphereBufferGeometry(202, 30, 30);
          let mat = new THREE.MeshPhongMaterial({
            map: texture,
            transparent: true,
            opacity: 1
            // emissive: 0xff0000
          });
          this.layersMap[param] = new THREE.Mesh(geo, mat);
          this.layersMap[param].visible = false;
          this.earthMesh.add(this.layersMap[param]);
          resolve(`${param}'s ok!!`);
        })
      });
    },

    // 监听选项
    setEventListener() {
      window.Bus.$on('close2D', (params) => {
        this.twoFlag = false;
        this.trackballControls.enabled = true;
        this.camera.position.set(-600, 300, -600);
      });

      window.Bus.$on('toggleCables', (cableInfo) => {
        console.log(cableInfo);
        if (this.layersMap[cableInfo.code]) {
          this.layersMap[cableInfo.code].visible = cableInfo.flag;
        } else {
          console.log(`-- Warn: There's no data about "${cableInfo.name}"!!`);
        }
      });

      window.Bus.$on('addSatellite', (info) => {
        if (info[0] === '_satellite') {
          this['satelliteMesh' + info[1]].visible = true;
          this['corn' + info[1]].visible = info[2];
          this['spotLight' + info[1]].visible = info[2];
          this['lightHelper' + info[1]].visible = info[2];
        } else if (info[0] === '_satelliteOrbit') {
          this.satelliteOrbit.visible = true;
        } else if (info[0] === '_satelliteCoverRegion') {
          if (this.layersMap[info[1]]) {
            this.layersMap[info[1]].visible = true;
          } else if (info[1] === '_12') {
            this.layersMap['_12_c'].visible = true;
            this.layersMap['_12_ku'].visible = true;
          } else {
            console.log(`-- Warn: There's no data about "${info[1]}"!!`);
          }
          this['corn' + info[1]].visible = info[2];
          this['spotLight' + info[1]].visible = info[2];
          this['lightHelper' + info[1]].visible = info[2];
        } else if (info[0] === '_satelliteStation') {
          // console.log(info);
          this['idrStationMesh' + info[1]].visible = true;
          this['superStationMesh' + info[1]].visible = true;
        }
      });

      window.Bus.$on('removeSatellite', (info) => {
        if (info[0] === '_satellite') {
          this['satelliteMesh' + info[1]].visible = false;
          this['corn' + info[1]].visible = info[2];
          this['spotLight' + info[1]].visible = info[2];
          this['lightHelper' + info[1]].visible = info[2];
        } else if (info[0] === '_satelliteOrbit') {
          this.satelliteOrbit.visible = false;
        } else if (info[0] === '_satelliteCoverRegion') {
          if (this.layersMap[info[1]]) {
            this.layersMap[info[1]].visible = false;
          } else if (info[1] === '_12') {
            this.layersMap['_12_c'].visible = false;
            this.layersMap['_12_ku'].visible = false;
          } else {
            console.log(`-- Warn: There's no data about "${info[1]}"!!`);
          }
          this['corn' + info[1]].visible = info[2];
          this['spotLight' + info[1]].visible = info[2];
          this['lightHelper' + info[1]].visible = info[2];
        } else if (info[0] === '_satelliteStation') {
          // console.log(info);
          this['idrStationMesh' + info[1]].visible = false;
          this['superStationMesh' + info[1]].visible = false;
        }
      });
    },

    // 暂时用不到
    open2D(flag) {
      this.camera.position.multiplyScalar(1.00001);
      this.dialogToOpenTwo = false;
      if (flag === 'yes') {
        this.trackballControls.enabled = false;
        this.twoFlag = true;
      }
    },

    // 添加虚线圆圈
    addDashedCircle() {
      var dashMat = new THREE.LineDashedMaterial({
        color: 0xee6666,
        dashSize: 5,
        gapSize: 5,
        linewidth: 16
      });
      let circGeo = new THREE.CircleGeometry(310, 100);
      circGeo.vertices.shift();
      circGeo.computeLineDistances();
      var circ = new THREE.Line(circGeo, dashMat);
      this.scene.add(circ);
    },

    // 添加卫星轨道
    addSatelliteOrbit() {
      let geo = new THREE.RingBufferGeometry(498, 502, 100);
      let mat = new THREE.MeshBasicMaterial({
        color: 0x2094fb,
        side: THREE.DoubleSide
      });
      let mesh = new THREE.Mesh(geo, mat);
      mesh.rotateX(Math.PI / 2);
      mesh.visible = false;
      this.scene.add(mesh);
      return mesh;
    },

    // 添加粒子系统
    createParticles() {
      var geom = new THREE.Geometry();
      // var geom = new THREE.SphereGeometry(500, 50, 50);
      var material = new THREE.PointsMaterial({
        // color: 0x00ffff,
        size: 4,
        vertexColors: true
      });
      var range = 600;
      for (var i = 0; i < 15000; i++) {
        var particle = new THREE.Vector3(Math.random() * range - range / 2, Math.random() * range - range / 2, Math.random() * range - range / 2);
        geom.vertices.push(particle);
        var color = new THREE.Color(0x00ff00);
        color.setHSL(color.getHSL().h, color.getHSL().s, Math.random() * color.getHSL().l);
        geom.colors.push(color);
      }
      let cloud = new THREE.Points(geom, material);
      cloud.sortParticles = true;
      this.scene.add(cloud);
    },

    // 创建同步卫星satellite，以及光锥、聚光灯、聚光灯辅助线；
    addSatellite() {
      return new Promise((resolve, reject) => {
        this.loader.load(`/myapi/img/satellite.png`, (texture) => {
          let geo = new THREE.PlaneBufferGeometry(60, 60, 1, 1);
          let mat = new THREE.MeshBasicMaterial({
            map: texture,
            transparent: true,
            side: THREE.DoubleSide
          });
          let posOf12 = this.lgltToXYZ(87.5, 0, 500);
          let posOf6a = this.lgltToXYZ(125, 0, 500);
          let posOf6b = this.lgltToXYZ(115.5, 0, 500);
          this.satelliteMesh_12 = new THREE.Mesh(geo, mat);
          this.satelliteMesh_6a = new THREE.Mesh(geo, mat);
          this.satelliteMesh_6b = new THREE.Mesh(geo, mat);
          this.satelliteMesh_12.position.copy(posOf12);
          this.satelliteMesh_6a.position.copy(posOf6a);
          this.satelliteMesh_6b.position.copy(posOf6b);
          this.satelliteMesh_12.visible = this.satelliteMesh_6a.visible = this.satelliteMesh_6b.visible = false;
          this.scene.add(this.satelliteMesh_12, this.satelliteMesh_6a, this.satelliteMesh_6b);
          this.satelliteFlag_12 = this.satelliteFlag_6a = this.satelliteFlag_6b = true;

          // 创建光锥
          this.corn_12 = this.addLightBeamFromSatellite(0xff0000, posOf12, this.scatterMesh.position);
          this.corn_6a = this.addLightBeamFromSatellite(0xff0000, posOf6a, this.scatterMesh.position);
          this.corn_6b = this.addLightBeamFromSatellite(0xff0000, posOf6b, this.scatterMesh.position);
          this.corn_12.visible = this.corn_6a.visible = this.corn_6b.visible = false;
          this.earthMesh.add(this.corn_12);
          this.earthMesh.add(this.corn_6a);
          this.earthMesh.add(this.corn_6b);

          // 创建光锥范围的聚光灯
          this.spotLight_12 = this.createSpotlight(0x00ffff);
          this.spotLight_6a = this.createSpotlight(0x00ffff);
          this.spotLight_6b = this.createSpotlight(0x00ffff);
          this.spotLight_12.position.copy(posOf12);
          this.spotLight_6a.position.copy(posOf6a);
          this.spotLight_6b.position.copy(posOf6b);
          this.spotLight_12.visible = this.spotLight_6a.visible = this.spotLight_6b.visible = false;
          this.scene.add(this.spotLight_12, this.spotLight_6a, this.spotLight_6b);

          // 创建光锥范围的聚光灯辅助线
          this.lightHelper_12 = new THREE.SpotLightHelper(this.spotLight_12);
          this.lightHelper_6a = new THREE.SpotLightHelper(this.spotLight_6a);
          this.lightHelper_6b = new THREE.SpotLightHelper(this.spotLight_6b);
          this.lightHelper_12.visible = this.lightHelper_6a.visible = this.lightHelper_6b.visible = false;
          this.scene.add(this.lightHelper_12, this.lightHelper_6a, this.lightHelper_6b);

          resolve('△△△创建同步卫星satellite，以及光锥、聚光灯、聚光灯辅助线!!!!!');
        });
      });
    },

    // 创建月球moon；
    addMoon() {
      return new Promise((resolve, reject) => {
        this.loader.load(`/myapi/img/moon-4k.jpg`, (texture) => {
          // console.time('kk');
          let geometry = new THREE.SphereBufferGeometry(30, 25, 25);
          let material = new THREE.MeshLambertMaterial();
          material.map = texture;
          // material.shininess = 6;
          this.moonMesh = new THREE.Mesh(geometry, material);
          this.moonMesh.position.set(0, 300, -1000);
          this.scene.add(this.moonMesh);
          this.moonFlag = true;
          // console.timeEnd('kk');
          resolve('△△△创建月球moon！！');
        })
      });
    },

    // 创建星空space；
    addSpace() {
      return new Promise((resolve, reject) => {
        this.loader.load(`/myapi/img/galaxy_starfield.png`, (texture) => {
          let geometry = new THREE.SphereBufferGeometry(4000, 10, 10);
          let material = new THREE.MeshBasicMaterial();
          material.map = texture;
          material.side = THREE.BackSide;
          this.spaceMesh = new THREE.Mesh(geometry, material);
          this.scene.add(this.spaceMesh);
          this.spaceFlag = true;
          resolve('△△△创建星空space！！');
        })
      });
    },

    // 创建圆锥体网孔，用来显示从卫星射向地球表面的光束；
    addLightBeamFromSatellite(color, position, target) {
      let coneGeo = new THREE.ConeBufferGeometry(21, 500, 50);
      let coneMat = new THREE.MeshBasicMaterial({
        color: color,
        transparent: true,
        opacity: 0.3,
        wireframe: false,
        side: THREE.DoubleSide
      });
      let cone = new THREE.Mesh(coneGeo, coneMat);
      // cone.position.set(0, 0, 500);
      cone.position.copy(position);
      cone.lookAt(target);
      cone.rotateX(-Math.PI / 2);
      cone.translateY(-250);
      return cone;
    },

    // 创建卫星之间的直线光束；
    addPipeBetweenSatellites(v1, v2) {
      // 因为Line网孔不能设置线宽-bug，所以还是用管道网孔来实现，路径用3维曲线模型-由2个向量来确定；
      let curve = new THREE.CatmullRomCurve3([v1, v2]);
      let geo = new THREE.TubeGeometry(
        curve, // path
        100, // segments
        10, // radius
        50, // radiusSegments
        false // closed
      );
      geo = new THREE.BufferGeometry().fromGeometry(geo);
      this.nMax2 = geo.attributes.position.count; // 保存顶点数量；
      let mat = new THREE.MeshLambertMaterial({
        color: 0xff0000,
        transparent: true,
        opacity: 0.6,
        side: THREE.DoubleSide
      }); // 材质对象
      // let mesh = new THREE.Mesh(geo, mat); // 管道网格模型对象；
      this.pipeBetweenSatellites = new THREE.Mesh(geo, mat); // 管道网格模型对象；
      this.pipeBetweenSatellites.geometry.setDrawRange(0, 0); // 动态画管道网孔；
      this.earthMesh.add(this.pipeBetweenSatellites);
    },

    // 创建地球表面的虚线路径；
    addDashedLineOnEarth(path) {
      var mat = new THREE.LineDashedMaterial({
        color: 0x00ffff,
        dashSize: 5,
        gapSize: 5,
        linewidth: 1
      });
      let curve = new THREE.CatmullRomCurve3(path); // 通过少数点来创建3维曲线；
      var geo = new THREE.Geometry();
      geo.vertices = curve.getPoints(100);
      geo.computeLineDistances();
      let circ = new THREE.Line(geo, mat);
      this.scene.add(circ);
      circ.position.set(300, 100, 0);
    },

    // 创建地球表面的通道；
    addPipeOnEarth(path) {
      // 根据3维曲线路径画管道网孔；
      let curve = new THREE.CatmullRomCurve3(path); // 通过少数点来创建3维曲线；
      this.curve = curve;
      let geo = new THREE.TubeGeometry( // 创建管道网孔；
        curve, // path
        50, // segments
        2, // radius
        5, // radiusSegments
        false // closed
      );
      geo = new THREE.BufferGeometry().fromGeometry(geo);
      this.nMax = geo.attributes.position.count; // 保存顶点数量；
      let mat = new THREE.MeshNormalMaterial({
        // color: 0xff0000,
        // transparent: true,
        // opacity: 0.5,
        // wireframe: true
        side: THREE.DoubleSide
      });
      this.pipeOnEarthMesh = new THREE.Mesh(geo, mat);
      this.pipeOnEarthMesh.geometry.setDrawRange(0, 0); // 动态画管道网孔；
      this.earthMesh.add(this.pipeOnEarthMesh);
    },

    // 地球表面画通道；
    addCable(point1, point2, color) {
      // 圆环面 THREE.TorusGeometry(radius, tube, radialSegments, tubularSegments, arc)
      let pos1 = this.lgltToXYZ(point1[0], point1[1], 199);
      let pos2 = this.lgltToXYZ(point2[0], point2[1], 199);
      // let center = new THREE.Vector3((pos1.x + pos2.x) / 2, (pos1.y + pos2.y) / 2, (pos1.z + pos2.z) / 2);
      let center = new THREE.Vector3().addVectors(pos1, pos2).divideScalar(2);

      let radius = pos1.distanceTo(pos2) / 2;
      let geo = new THREE.TorusBufferGeometry(radius, 1, 10, 80, Math.PI * 2);
      geo.lookAt(pos1.cross(pos2));
      let mat = new THREE.MeshBasicMaterial({
        // color: 0xf88fa7,
        color: color,
        // transparent: true,
        // opacity: 0.8,
        wireframe: true
      });
      let mesh = new THREE.Mesh(geo, mat);
      mesh.position.copy(center);
      this.earthMesh.add(mesh);
      return mesh;
    },

    // 地球表面打散点；
    addScatter(lon, lat, color) {
      // scatter
      let scatterGeo = new THREE.SphereBufferGeometry(5, 16, 16);
      let scatterMat = new THREE.MeshBasicMaterial({
        color: color || 0xff0000
      });
      let scatterMesh = new THREE.Mesh(scatterGeo, scatterMat);
      let position = this.lgltToXYZ(lon, lat, 200);
      scatterMesh.position.set(position.x, position.y, position.z);

      // let label = Math.random().toFixed(2);
      // console.time(label);
      this.earthMesh.add(scatterMesh);
      // console.timeEnd(label);
      // scene.add(scatterMesh);
      return scatterMesh;
    },

    // 地球表面添加基站；
    addCircleOnEarthSurface(isIdr, data, color = 0xff0000) {
      // circle
      let pic = isIdr ? 'idr-station.png' : 'super-station.png';
      this.loader.load(`/myapi/img/${pic}`, (texture) => {
        // texture.anisotropy = this.renderer.getMaxAnisotropy(); // 设置纹理的各向异性为GPU的最大有效值；
        let geo = new THREE.PlaneGeometry(8, 8);
        let mat = new THREE.MeshBasicMaterial({
          map: texture,
          side: THREE.DoubleSide,
          // opacity: 0.4,
          transparent: true
          // emissive: 0xff0000
        });

        let geoGroup12 = new THREE.Geometry();
        let geoGroup6b = new THREE.Geometry();
        let radius = [];
        for (let i = 200; i < 250; i += 0.14) {
          radius.push(i);
        }
        console.log(data);
        // let radius = [201, 201.5, 202, 202.5, 203, 203.5, 204, 204.5, 205, 205.5, 206, 206.5, 207, 207.5, 208, 208.5, 209, 209.5, 210, 210.5, 211, 211.5, 212, 212.5, 213, 213.5, 214, 214.5, 215, 215.5];
        for (let x in data) {
          let mesh = new THREE.Mesh(geo, mat);
          if (data[x][0].LONGITUDE) {
            data[x].forEach((item, index) => {
              let pos = this.lgltToXYZ(item.LONGITUDE, item.LATITUDE, radius[index % radius.length]);
              mesh.position.set(pos.x, pos.y, pos.z);
              mesh.lookAt(this.earthMesh.position);
              mesh.updateMatrix();
              if (item.SATELLITE === '中星12号') {
                geoGroup12.merge(mesh.geometry, mesh.matrix);
              } else {
                geoGroup6b.merge(mesh.geometry, mesh.matrix);
              }
            })
            // for (let y of data[x]) {
            //   let pos = this.lgltToXYZ(y.LONGITUDE, y.LATITUDE, 200 + 50 * Math.random());
            //   mesh.position.set(pos.x, pos.y, pos.z);
            //   mesh.lookAt(this.earthMesh.position);
            //   mesh.updateMatrix();
            //   geoGroup.merge(mesh.geometry, mesh.matrix);
            // }
          }
        }
        if (isIdr) {
          this.idrStationMesh_12 = new THREE.Mesh(geoGroup12, mat);
          this.idrStationMesh_6b = new THREE.Mesh(geoGroup6b, mat);
          // this.idrStationMesh_12.visible = false;
          // this.idrStationMesh_6b.visible = false;
          this.earthMesh.add(this.idrStationMesh_12, this.idrStationMesh_6b);
        } else {
          this.superStationMesh_12 = new THREE.Mesh(geoGroup12, mat);
          this.superStationMesh_6b = new THREE.Mesh(geoGroup6b, mat);
          // this.superStationMesh_12.visible = false;
          // this.superStationMesh_6b.visible = false;
          this.earthMesh.add(this.superStationMesh_12, this.superStationMesh_6b);
        }
        // let meshGroup = new THREE.Mesh(geoGroup, mat);
        // this.earthMesh.add(meshGroup);
      })

      // let geo = new THREE.CircleGeometry(1, 32);
      // let mat = new THREE.MeshBasicMaterial({
      //   color: color,
      //   side: THREE.DoubleSide,
      //   opacity: 0.4,
      //   transparent: true
      // });
    },

    // 地球表面打图形自定义散点；
    addBaseStation(lon, lat) {
      return new Promise((resolve, reject) => {
        this.loader.load(`/myapi/img/satellite.png`, (texture) => {
          let baseStationGeo = new THREE.PlaneBufferGeometry(20, 20, 1, 1);
          let baseStationMat = new THREE.MeshBasicMaterial({
            map: texture,
            transparent: true,
            side: THREE.DoubleSide
          });
          let baseStationMesh = new THREE.Mesh(baseStationGeo, baseStationMat);
          let pos = this.lgltToXYZ(lon, lat, 205);
          baseStationMesh.position.set(pos.x, pos.y, pos.z);
          // let label = Math.random().toFixed(2);
          // console.time(label);
          this.earthMesh.add(baseStationMesh);
          // console.timeEnd(label);
          baseStationMesh.lookAt(new THREE.Vector3(0, 0, 0));
          this.baseStationArr.push(baseStationMesh);
          resolve('地球表面打图形自定义散点');
        });
      });
    },

    // 地球经纬度坐标 → XYZ笛卡尔坐标；
    lgltToXYZ(lon, lat, r) {
      lon = lon + 90;
      let x = r * Math.cos(2 * Math.PI * lat / 360) * Math.sin(2 * Math.PI * lon / 360);
      let y = r * Math.sin(2 * Math.PI * lat / 360);
      let z = r * Math.cos(2 * Math.PI * lat / 360) * Math.cos(2 * Math.PI * lon / 360);
      return new THREE.Vector3(x, y, z);
    },

    // 添加光源；
    initLight() {
      // this.light2 = new THREE.AmbientLight(0xffffff, 0.2);
      this.light2 = new THREE.AmbientLight(0xffffff, 1.1);
      this.scene.add(this.light2);

      // this.sunLight = new THREE.DirectionalLight(0xffffff, 0.3);
      // this.scene.add(this.sunLight);
      // this.sunHelper = new THREE.DirectionalLightHelper(this.sunLight);
      // this.scene.add(this.sunHelper);

      // sunLight = new THREE.PointLight(0xffffd8, 1, 0);
      // sunLight.position.set(40000, 0, -4000);
      // scene.add(sunLight);
      //
      // sunLight = new THREE.SpotLight(0xffffff, 2);
      // sunLight.castShadow = true;
      // sunLight.intensity = 3;
      // sunLight.angle = 0.3;
      // sunLight.penumbra = 0.8;
      // sunLight.decay = 0.1;
      // sunLight.distance = 3000;
      // sunLight.shadow.mapSize.width = 1024;
      // sunLight.shadow.mapSize.height = 1024;
      // sunLight.position.set(0, 300, -1000);
      // scene.add(sunLight);
      // sunHelper = new THREE.SpotLightHelper(sunLight);
      // scene.add(sunHelper);
    },

    // 初始化空间场景；
    initScene() {
      this.scene = new THREE.Scene();
    },

    // 初始化渲染器；
    initRenderer() {
      this.width = document.getElementById('canvas-frame').clientWidth;
      this.height = document.getElementById('canvas-frame').clientHeight;
      this.renderer = new THREE.WebGLRenderer({
        antialias: true
      });
      this.renderer.setSize(this.width, this.height);
      document.getElementById('canvas-frame').appendChild(this.renderer.domElement);
      this.renderer.setClearColor(0x000000, 1.0);

      this.stats = new Stats.Stats();
      this.stats.domElement.style.position = 'absolute';
      this.stats.domElement.style.left = '0px';
      this.stats.domElement.style.top = '0px';
      this.stats.domElement.style.zIndex = 1;
      document.getElementById('canvas-frame').appendChild(this.stats.domElement);
    },

    // 初始化摄像机；
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(50, this.width / this.height, 1, 8000);
      this.camera.position.set(-800, 200, -300);
      // this.camera.position.set(-80, 280, -400);
      this.camera.up.x = 0;
      this.camera.up.y = 1;
      this.camera.up.z = 0;
      this.camera.lookAt({
        x: 0,
        y: 0,
        z: 0
      });

      this.clock = new THREE.Clock();
      // this.orbitControls = new THREE.OrbitControls(this.camera);
      // this.orbitControls.userZoomSpeed = 0.3;
      // this.orbitControls.minDistance = 400;
      // this.orbitControls.maxDistance = 4000;

      this.trackballControls = new THREE.TrackballControls(this.camera);
      this.trackballControls.rotateSpeed = 1;
      this.trackballControls.zoomSpeed = 1;
      this.trackballControls.staticMoving = true;
      this.trackballControls.minDistance = 100;
      this.trackballControls.maxDistance = 4000;
    },

    addLayerForCableAndCoverRegion() {  // 添加缆线和覆盖范围的图层
      return new Promise((resolve, reject) => {
        let promiseArr = [];
        for (let key in this.layersMap) {
          promiseArr.push(this.addLayer(key));
        }
        Promise.all(promiseArr).then((res) => {
          console.log(res);
          this.setEventListener();
          resolve('△△△添加所有图层（缆线、覆盖范围）!!');
        });
      })
    },

    addFont() {  // 添加文字
      return new Promise((resolve, reject) => {
        // 加载字体
        var loader = new THREE.FontLoader(); // 实例化字体包加载器；
        loader.load(`/myapi/font/FZLanTingHeiS-L-GB_Regular.json`, (font) => { // 加载字体包，并回调；
          var textOfPerson = new THREE.Mesh(new THREE.TextGeometry('WEBGL-SHADERS', {
            font: font,
            size: 5,
            height: 1
          }), new THREE.MeshBasicMaterial({
            color: 0xff0000
          }));
          // textOfPerson.position.set(200, 200, 300);

          let pos = this.lgltToXYZ(100 - 1, 30 - 1, 251);
          textOfPerson.position.set(pos.x, pos.y, pos.z);

          textOfPerson.lookAt(this.earthMesh.position);
          textOfPerson.rotateY(Math.PI);
          // textOfPerson.translateX(-150);
          this.earthMesh.add(textOfPerson);

          resolve('△△△添加文字!!');
        })
      })
    },

    addStation() {  // 添加基站打点（IDR基站:jt0001; 超级基站:jt0002;）
      return new Promise((resolve, reject) => {
        let idr = new Promise((resolve, reject) => {
          this.$http.indi.get('jt0001', {}, (res) => {
            console.log(res);
            resolve(res.data.data);
          });
        });
        let sup = new Promise((resolve, reject) => {
          this.$http.indi.get('jt0002', {}, (res) => {
            resolve(res.data.data);
          });
        });
        Promise.all([idr, sup]).then((res) => {
          console.log(res);
          res.forEach((data, index) => {
            let province = [];
            for (let x of data) {
              province.push(x.PROVINCE);
            }
            province = Array.from(new Set(province)); // 地市去重
            let obj = {};
            for (let x of province) {
              obj[x] = data.filter((item) => {
                return item.PROVINCE === x;
              })
            }
            console.log(obj);

            for (let x in obj) {
              if (obj[x].length >= 100) {
                obj[x] = [chinaMap[x][0], chinaMap[x][1], obj[x].length];
              }
            }
            console.log(obj);
            let isIdr = index === 0;
            this.addCircleOnEarthSurface(isIdr, obj);

            resolve('△△△添加基站打点-' + index === 0 ? 'IDR基站' : '超级基站');
          })
        });
      });
    },

    addShapeOf2D() {  // 添加任意形状的2D图形
      let rectLength = 60;
      let rectWidth = 20;
      let rectShape = new THREE.Shape();
      rectShape.moveTo(200, 200);
      rectShape.lineTo(200, 200 + rectWidth);
      rectShape.lineTo(200 + rectLength, 200 + rectWidth);
      rectShape.lineTo(200 + 60, 200 + 30);
      rectShape.lineTo(200 + 100, 200 + 30);
      rectShape.lineTo(200 + rectLength, 200 + 0);
      rectShape.lineTo(200 + 0, 200 + 0);
      let rectGeom = new THREE.ShapeGeometry(rectShape);
      let rectMesh = new THREE.Mesh(rectGeom, new THREE.MeshBasicMaterial({
        color: 0xffffff,
        transparent: true,
        opacity: 0.4,
        side: THREE.DoubleSide
      }));
      rectMesh.position.set(0, 0, 200);
      // rectMesh.lookAt(new THREE.Vector3(0, 0, 0));
      // for (let x of rectGeom.vertices) {
      //   x.setLength(200);
      // }
      // rectMesh.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 3);
      this.scene.add(rectMesh);
    },

    // 创建网孔
    initObject() {
      // 创建网格；
      let helper = new THREE.GridHelper(2000, 10);
      // helper.setColors( 0x0000ff, 0xffffff );
      this.scene.add(helper);

      this.loader = new THREE.TextureLoader(); // 创建纹理加载器；

      this.addEarth().then((res) => { // 创建地球earth；
        console.log(res);

        this.addLayerForCableAndCoverRegion().then((res) => {  // 添加缆线和卫星覆盖范围的图层；
          console.log(res);
        });

        // this.addStation().then((res) => {  // 添加基站打点；
        //   console.log(res);
        // });

        // 使用本地基站数据
        for (let key in stationInfo) {
          let province = [];
          for (let x of stationInfo[key]) {
            province.push(x.PROVINCE);
          }
          province = Array.from(new Set(province)); // 地市去重
          let obj = {};
          for (let x of province) {
            obj[x] = stationInfo[key].filter((item) => {
              return item.PROVINCE === x;
            })
          }
          // for (let x in obj) {
          //   if (obj[x].length >= 100) {
          //     obj[x] = [chinaMap[x][0], chinaMap[x][1], obj[x].length];
          //   }
          // }
          let isIdr = key === 'idr';
          this.addCircleOnEarthSurface(isIdr, obj);
        }

        this.addFont().then((res) => {  // 创建文字；
          console.log(res);
        });

        this.addMoon().then((res) => {  // 创建月球moon；
          console.log(res);
        });

        this.addSpace().then((res) => {  // 创建星空space；
          console.log(res);
        });

        this.addSatellite().then((res) => { // 创建同步卫星，以及光锥、聚光灯、聚光灯辅助线
          console.log(res);
        });

        this.satelliteOrbit = this.addSatelliteOrbit(); // 添加卫星轨道

        // // 加载字体
        // var loader = new THREE.FontLoader(); // 实例化字体包加载器；
        // loader.load('http://localhost:3000/FZLanTingHeiS-L-GB_Regular.json', (font) => { // 加载字体包，并回调；
        //   var textOfPerson = new THREE.Mesh(new THREE.TextGeometry('WEBGL-SHADERS', {
        //     font: font,
        //     size: 5,
        //     height: 1
        //   }), new THREE.MeshBasicMaterial({
        //     color: 0xffff00
        //   }));
        //   // textOfPerson.position.set(200, 200, 300);
        //
        //   let pos = this.lgltToXYZ(100 - 1, 30 - 1, 201);
        //   textOfPerson.position.set(pos.x, pos.y, pos.z);
        //
        //   textOfPerson.lookAt(this.earthMesh.position);
        //   textOfPerson.rotateY(Math.PI);
        //   // textOfPerson.translateX(-150);
        //   this.earthMesh.add(textOfPerson);
        // })

        this.addShapeOf2D(); // 添加卫星轨道
        // let rectLength = 60;
        // let rectWidth = 20;
        // let rectShape = new THREE.Shape();
        // rectShape.moveTo(200, 200);
        // rectShape.lineTo(200, 200 + rectWidth);
        // rectShape.lineTo(200 + rectLength, 200 + rectWidth);
        // rectShape.lineTo(200 + 60, 200 + 30);
        // rectShape.lineTo(200 + 100, 200 + 30);
        // rectShape.lineTo(200 + rectLength, 200 + 0);
        // rectShape.lineTo(200 + 0, 200 + 0);
        // let rectGeom = new THREE.ShapeGeometry(rectShape);
        // let rectMesh = new THREE.Mesh(rectGeom, new THREE.MeshBasicMaterial({
        //   color: 0xffffff,
        //   transparent: true,
        //   opacity: 0.4,
        //   side: THREE.DoubleSide
        // }));
        // rectMesh.position.set(0, 0, 200);
        // // rectMesh.lookAt(new THREE.Vector3(0, 0, 0));
        // // for (let x of rectGeom.vertices) {
        // //   x.setLength(200);
        // // }
        // // rectMesh.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 3);
        // this.scene.add(rectMesh);

        let path = []; // 存放曲线路径的顶点向量；
        for (let i = 0; i < 90; i++) {
          path.push(this.lgltToXYZ(20 + i, 0 + i, 300));
        }
        // path[45].x = path[46].x + 20;
        // path[45].z = path[46].z + 20;
        this.addPipeOnEarth(path); // 创建地球表面的通道；

        this.addDashedLineOnEarth(path); // 创建地球表面的虚线路径；

        // 创建卫星之间的直线光束；
        this.addPipeBetweenSatellites(new THREE.Vector3(300, 100, 200), new THREE.Vector3(400, 1000, 200));

        // 用于测试，用来设定圆锥体和聚光灯的朝向；
        this.scatterMesh = this.addScatter(117, 22);

        // 创建地球表面的任意经纬度包围区域；
        this.$http.indi.get('nmg.json', null, (res) => {
          // console.log(res.data[0].KPI_VALUE1);
          res.data[0].KPI_VALUE1.sort((a, b) => {
            return a[0] - b[0];
          });
          let arr = res.data[0].KPI_VALUE1;
          let points = [];
          for (let x = 0; x < arr.length; x += 2000) {
            // this.addScatter(arr[x][0], arr[x][1], Math.random() * 0xffffff)
            points.push(this.lgltToXYZ(arr[x][0], arr[x][1], 301));
          }
          points.push(new THREE.Vector3(0, 0, 0));
          points.push(new THREE.Vector3(100, 300, -320));
          geo = new THREE.ConvexBufferGeometry(points);
          mat = new THREE.MeshBasicMaterial({
            color: 0xff0000,
            wireframe: true
            // transparent: true,
            // opacity: 0.5
          });
          let convexMesh = new THREE.Mesh(geo, mat);
          this.earthMesh.add(convexMesh);
        })

        // 随机在地球表面打散点；
        // for (let i = 0; i < 20; i++) {
        //   this.addScatter(-180 + 360 * Math.random(), -40 + 80 * Math.random(), Math.random() * 0xffffff)
        // }

        // 随机在地球表面打散点，并在2个散点间形成半圆形通道；
        // for (let i = 0; i < 20; i++) {
        //   // let point1 = [110, 20];
        //   // let point2 = [-10, 10];
        //   let point1 = [-180 + 360 * Math.random(), -40 + 80 * Math.random()];
        //   let point2 = [-180 + 360 * Math.random(), -40 + 80 * Math.random()];
        //   this.addScatter(point1[0], point1[1], 0xffffff);
        //   this.addScatter(point2[0], point2[1], 0xffffff);
        //   this.cables.push(this.addCable(point1, point2, Math.random() * 0xffffff));
        // }

        // 在地球表面打图形自定义散点；
        let arr = [
          [150, 41],
          [160, 42],
          [170, 43],
          [180, 44],
          [190, 45]
        ];
        for (let x of arr) {
          this.addBaseStation(x[0], x[1]).then((res) => {
            console.log(res);
          });
        }

        // this.addDashedCircle(); // 创建虚线圆环；
        // this.createParticles(); // 创建粒子；
      });
    },

    // 每帧的重绘函数
    renderRecursion() {
      let delta = this.clock.getDelta();
      // console.log(delta);
      this.trackballControls.update(delta);
      // this.orbitControls.update(delta);

      let cameraLength = this.camera.position.length();
      this.distance = +cameraLength.toFixed(3);
      if (cameraLength <= 100) {
        this.dialogToOpenTwo = true;
        // this.trackballControls.enabled = false;
      }

      if (this.earthFlag && this.spaceFlag && this.moonFlag && this.satelliteFlag_12 && this.satelliteFlag_6a && this.satelliteFlag_6b) {
        this.earthSpin += 0.001;
        // this.earthSpin += delta * 0.1;
        this.satelliteMesh_12.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.satelliteMesh_6a.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.satelliteMesh_6b.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        // this.satelliteMesh.position.set(500 * Math.sin(this.earthSpin), 0, 500 * Math.cos(this.earthSpin));
        // this.satelliteMesh.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.satelliteMesh_12.lookAt(this.camera.position);
        this.satelliteMesh_6a.lookAt(this.camera.position);
        this.satelliteMesh_6b.lookAt(this.camera.position);

        // spotLight2.target.position.set(1766, 684, -642);
        // spotLight2.target.updateMatrixWorld();
        // this.spotLight_12.rotateX(0.01);
        this.spotLight_12.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.spotLight_12.target = this.scatterMesh;
        this.lightHelper_12.update();

        this.spotLight_6a.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.spotLight_6a.target = this.scatterMesh;
        this.lightHelper_6a.update();

        this.spotLight_6b.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.001);
        this.spotLight_6b.target = this.scatterMesh;
        this.lightHelper_6b.update();

        this.earthMesh.rotation.y = this.earthSpin;

        this.moonMesh.rotation.y = this.earthSpin * 4;
        this.moonMesh.position.set(1000 * Math.sin(-this.earthSpin / 1), 300, -1000 * Math.cos(-this.earthSpin / 1));

        // this.sunLight.position.set(5000 * Math.sin(this.earthSpin * 4), 1000, -5000 * Math.cos(this.earthSpin * 4));
        // this.sunLight.target = this.earthMesh;
        // this.sunHelper.update();

        this.spaceMesh.rotation.x = this.earthSpin / 4;
        this.spaceMesh.rotation.y = this.earthSpin / 4;
        this.spaceMesh.rotation.z = this.earthSpin / 4;

        if (this.nEnd < this.nMax) {
          this.nEnd += this.nStep;
          this.pipeOnEarthMesh.geometry.setDrawRange(0, this.nEnd); // 动态画管道网孔；
        }
        if (this.nEnd2 < this.nMax2) {
          this.nEnd2 += this.nStep2;
          this.pipeBetweenSatellites.geometry.setDrawRange(0, this.nEnd2); // 动态画管道网孔；
        }

        // this.scene.rotateX(0.001);

        // 月球网孔的顶点波浪式运动
        let time = +new Date();
        for (var i = 2, l = this.earthMesh.geometry.attributes.position.array.length; i < l; i += 3) {
          this.earthMesh.geometry.attributes.position.array[i] = 15 * Math.sin(i / 100 + time / 200);
        }
        let vertices = new THREE.BufferAttribute(this.earthMesh.geometry.attributes.position.array, 3);
        this.earthMesh.geometry.addAttribute('position', vertices);

        // // 管道穿越
        // this.camera.position.copy(this.curve.getPoint(this.earthSpin * 2 % 1));
        // this.camera.lookAt(this.curve.getPoint(this.earthSpin * 2 % 1 + 0.05));
        // this.camera.rotateZ(this.earthSpin * 10);
      }

      this.stats.update();
      // TWEEN.update(); // 启动TWEEN动画补间；
      this.renderer.render(this.scene, this.camera);
      this.renderTimer = requestAnimationFrame(this.renderRecursion);
    }
  },
  beforeDestroy() {
    window.cancelAnimationFrame(this.renderTimer);
  },
  components: {
    Two
  }
}
</script>
