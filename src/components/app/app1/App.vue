<template>
<app-template class="a1" appName="matrix">
  <!-- <selection></selection> -->
  <three></three>
</app-template>
</template>
<script>
import './assets/scss/app.scss'
import Three from './views/3d/ThreeMain.vue'
import Selection from './views/selection/SelectionMain.vue'

export default {
  data() {
    return {
      // currentView: 'Three'
    }
  },
  mounted() {
    this.$nextTick(() => {
      // window.Bus.$on('viewSwitch', (params) => {
      //   if (params === 'to2D') {
      //     this.currentView = 'Two';
      //   } else if (params === 'to3D') {
      //     this.currentView = 'Three';
      //   }
      // });
    });
  },
  methods: {

  },
  components: {
    Selection,
    Three
  }
}
</script>

<style>
@font-face {
  font-family: 'DigifaceWide';
  src: url('../../../assets/fonts/DigifaceWide.ttf');
}

@font-face {
  font-family: 'PixelLCD-7';
  src: url('../../../assets/fonts/PixelLCD-7.ttf')
}

.bounce-enter-active {
  animation: bounce-in .6s;
}

.bounce-leave-active {
  animation: bounce-out .4s;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
    margin-left: 100%;
  }
  /*50% {
          transform: scale(1.5);
        }*/
  100% {
    margin-left: 0;
    transform: scale(1);
  }
}

@keyframes bounce-out {
  0% {
    margin-left: 0;
    transform: scale(1);
  }
  /*50% {
          transform: scale(1.5);
        }*/
  100% {
    margin-left: -100%;
    transform: scale(0);
  }
}
</style>
