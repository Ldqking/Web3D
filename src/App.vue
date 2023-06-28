<script>
import * as THREE from "three";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import Stats from "three/examples/jsm/libs/stats.module";

export default {
  data() {
    return {
      // scene: "",
      light: "",
      camera: "",
      controls: "",
      renderer: "",
      stats: '',
      clock:'',
      mixer: null
    }
  },
  mounted() {
    // 创建一个时钟对象Clock
    this.clock = new THREE.Clock();

    this.init();
    this.animate();
  },
  methods: {
    init() {
      let that = this;
      let container = document.getElementById("threeContained");
      that.camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 2000 );
      that.camera.position.set( 0, 300, 1000 );

      that.$scene = new THREE.Scene();
      that.$scene.background = new THREE.Color( 0xa0a0a0 );

      const hemiLight = new THREE.HemisphereLight( 0xffffff, 0x444444, 5 );
      hemiLight.position.set( 0, 200, 0 );
      that.$scene.add( hemiLight );

      const loader = new FBXLoader();
				loader.load( '/Web3D/model/fbx/test.fbx', function ( obj ) {
          let object = obj
          console.log(object);

					that.mixer = new THREE.AnimationMixer( object );

					// const action = that.mixer.clipAction( object.animations[ 0 ] );
					// action.play();

					object.traverse( function ( child ) {

						if ( child.isMesh ) {

							child.castShadow = true;
							child.receiveShadow = true;

						}

					} );

					that.$scene.add( object );
				} );
        that.renderer = new THREE.WebGLRenderer( { antialias: true } );
				that.renderer.setPixelRatio( window.devicePixelRatio );
				that.renderer.setSize( window.innerWidth, window.innerHeight );
				that.renderer.useLegacyLights = false;
				that.renderer.shadowMap.enabled = false;
				container.appendChild( that.renderer.domElement );

				const controls = new OrbitControls( that.camera, that.renderer.domElement );
				controls.target.set( 0, 100, 0 );
				controls.update();

				window.addEventListener( 'resize', this.onWindowResize );

				// stats
				that.stats = new Stats();
				container.appendChild( that.stats.dom );
    },
    onWindowResize() {
				this.camera.aspect = window.innerWidth / window.innerHeight;
				this.camera.updateProjectionMatrix();
				this.renderer.setSize( window.innerWidth, window.innerHeight );
		},
    animate() {

				requestAnimationFrame( this.animate );

				const delta = this.clock.getDelta();

				if ( this.mixer ) this.mixer.update( delta );

				this.renderer.render( this.$scene, this.camera );

				this.stats.update();

			}
  }
}
</script>

<template>
  <div id="threeContained"></div>
</template>

<style scoped>
.three-canvas,#threeContained {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #d6eaff;
}
</style>
