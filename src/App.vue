<script>
import * as THREE from 'three'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader' //fbx模型加载器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls' // 轨道控制器
import Stats from 'three/examples/jsm/libs/stats.module' // 性能监视器

export default {
  data() {
    return {
      // scene: "",
      light: '',
      camera: '',
      controls: '',
      renderer: '',
      stats: '',
      clock: '',
      mixer: null
    }
  },
  mounted() {
    // 创建一个时钟对象Clock
    this.clock = new THREE.Clock()

    this.init()
    this.animate()
  },
  methods: {
    init() {
      let that = this
      // canvas容器
      let container = document.getElementById('threeContained')
      // 定义透视相机
      // 参考：https://threejs.org/docs/index.html#api/zh/cameras/PerspectiveCamera
      that.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 2000)
      // 设置相机位置
      // 参考：https://juejin.cn/post/7058048349584392200
      that.camera.position.set(0, 300, 1000)
      // 初始化一个场景
      that.$scene = new THREE.Scene()
      // 场景背景颜色
      that.$scene.background = new THREE.Color(0xa0a0a0)
      // 添加半球光线
      // 参考：https://threejs.org/docs/index.html#api/zh/lights/HemisphereLight
      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 5)
      hemiLight.position.set(0, 200, 0)
      that.$scene.add(hemiLight)
      // 实例化fbx加载器，加载模型
      const loader = new FBXLoader()
      // vue项目，模型素材放在public目录下（部署到github.io模型加载路径前'/Web3D',否则找不到静态资源）
      loader.load('/model/fbx/test.fbx', function (obj) {
        let object = obj
        // console.log(object)
        // 定义动画混合器
        // 参考：https://threejs.org/docs/index.html#api/zh/animation/AnimationMixer
        that.mixer = new THREE.AnimationMixer(object)

        // const action = that.mixer.clipAction( object.animations[ 0 ] );
        // action.play();
        // object.traverse( function ( child ) {
        // 	if ( child.isMesh ) {
        // 		child.castShadow = true;
        // 		child.receiveShadow = true;
        // 	}
        // } );
        // 把模型添加到场景中
        that.$scene.add(object)
      })
      // 实例化webGL渲染器
      // 参考：https://threejs.org/docs/index.html#api/zh/renderers/WebGLRenderer
      that.renderer = new THREE.WebGLRenderer({ antialias: true })
      // 设置设备像素比。通常用于避免HiDPI设备上绘图模糊
      that.renderer.setPixelRatio(window.devicePixelRatio)
      // 将输出canvas的大小调整为(width, height)并考虑设备像素比，且将视口从(0, 0)开始调整到适合大小
      that.renderer.setSize(window.innerWidth, window.innerHeight)
      // 是否使用传统照明模式。默认是true
      that.renderer.useLegacyLights = false
      // 如果使用，它包含阴影贴图的引用。
      // enabled: 如果设置开启，允许在场景中使用阴影贴图。默认是 false。
      that.renderer.shadowMap.enabled = false
      // 将渲染出来的dom添加到容器中
      container.appendChild(that.renderer.domElement)
      // 实例化轨道控制器
      // 参考：https://threejs.org/docs/index.html#examples/zh/controls/OrbitControls
      const controls = new OrbitControls(that.camera, that.renderer.domElement)
      // 控制器的焦点，.object的轨道围绕它运行。 它可以在任何时候被手动更新，以更改控制器的焦点。
      controls.target.set(0, 100, 0)
      // 更新控制器。必须在摄像机的变换发生任何手动改变后调用
      controls.update()
      // 监听窗口改变事件
      window.addEventListener('resize', this.onWindowResize)

      // 实例化一个性能监视器stats
      that.stats = new Stats()
      container.appendChild(that.stats.dom)
    },
    onWindowResize() {
      // 窗口改变重新设置摄像机视锥体的长宽比，通常是使用画布的宽/画布的高
      this.camera.aspect = window.innerWidth / window.innerHeight
      // 更新摄像机投影矩阵。在任何参数被改变以后必须被调用。
      this.camera.updateProjectionMatrix()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
    },
    animate() {
      // 执行一个动画，并且要求浏览器在下次重绘之前调用animate()更新动画
      requestAnimationFrame(this.animate)
      // 获取自 .oldTime 设置后到当前的秒数。
      // 参考；https://threejs.org/docs/index.html?q=clo#api/zh/core/Clock
      const delta = this.clock.getDelta()
      if (this.mixer) this.mixer.update(delta)
      // 渲染场景
      this.renderer.render(this.$scene, this.camera)
      // 性能监视器更新
      this.stats.update()
    }
  }
}
</script>

<template>
  <div id="threeContained"></div>
</template>

<style scoped>
.three-canvas,
#threeContained {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #d6eaff;
}
</style>
