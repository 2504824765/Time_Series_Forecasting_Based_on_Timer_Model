<script setup lang="ts">
import { ref, reactive, nextTick, watch } from "vue";
import BorderBox13 from "@/components/datav/border-box-13";
import * as THREE from 'three';
import { onMounted } from "vue";
import { useSettingStore } from "@/stores";
import  { GLTFLoader } from "three/addons/loaders/GLTFLoader.js"; // gltf模型引入
import { OrbitControls } from "three/addons/controls/OrbitControls.js"; // 坐标轴控制器
import { CSS2DRenderer,CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js'; // CSS2标签
import { CSS3DRenderer,CSS3DObject } from 'three/addons/renderers/CSS3DRenderer.js';

// 以下为常用调整变量
let model_scale = 3.8 // 发动机模型的大小

withDefaults(
    defineProps<{
      // 结束数值
      title: number | string;
    }>(),
    {
      title: "地图",
    }
);

var areas = [{
  name: "中国",
  position: [116.20, 39.55]
}, {
  name: "中非共和国",
  position: [18.35, 4.23]
}, {
  name: "智利",
  position: [-70.40, -33.24]
}, {
  name: "乍得",
  position: [14.59, 12.10]
}, {
  name: "赞比亚",
  position: [28.16, -15.28]
}, {
  name: "越南",
  position: [105.55, 21.05]
}, {
  name: "约旦",
  position: [35.52, 31.57]
}, {
  name: "维尔京群岛",
  position: [-64.37, 18.27]
}, {
  name: "英国",
  position: [-0.05, 51.36]
}];

const settingStore = useSettingStore();
const scene = new THREE.Scene(); // 创建一个场景
// 设置相机
const camera = new THREE.PerspectiveCamera(
    30,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
let speed = ref(0.005) // 存放旋转速度
let screenDom = ref(null) // 获取dom

// 动态更新旋转速度
watch(() => settingStore.indexConfig.modelRotateSpeed, (newVal) => {
  speed.value = newVal ? 0.005 : 0;
});

// 初始化模型
function initThree() {
  scene.background = null; // 设置场景背景为透明

  // 创建辅助坐标轴
  // let axesHelper = new THREE.AxesHelper(5)  // 设置坐标轴大小
  // scene.add(axesHelper)

  const threeDemo = document.getElementById("dom");
  const renderer = new THREE.WebGLRenderer({
    canvas: threeDemo,
    antialias: true,
    alpha: true, // 设置渲染器的 alpha 通道为 true
    logarithmicDepthBuffer: false, //如果要在单个场景中处理巨大的比例差异，就有必要使用。此处要用false，否则文字标签不显示
  });


  camera["position"].z = 30;
  camera["position"].y = -10;

  // 在页面中添加标签
  const div = document.getElementById('tag');
  const tag = new CSS2DObject(div);
  tag.position.set(3,3,3);
  const group = new THREE.Group();
  group.add(tag);

  function animate() {
    renderer.render(scene, camera);
    requestAnimationFrame(animate);


  }

  // 添加轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update()
  animate();

  // 添加标签
  createAreaPoint();
  renderScene();

  // 创建标签
  function createAreaPoint() {
    // 循环创建标签
    for (let i = 0, length = areas.length; i < length; i++) {
      let name = areas[i].name
      let position = createPosition(areas[i].position)
      createTxt(position,name); // 精灵标签函数
    }
  }

  // 经纬度转坐标
  function createPosition(lnglat:any) {
    let spherical = new THREE.Spherical
    spherical.radius = 10;
    let lng = lnglat[0]
    let lat = lnglat[1]
    let theta = (lng + 90) * (Math.PI / 180)
    let phi = (90 - lat) * (Math.PI / 180)
    spherical.phi = phi; // phi是方位面（水平面）内的角度，范围0~360度
    spherical.theta = theta; // theta是俯仰面（竖直面）内的角度，范围0~180度
    let position = new THREE.Vector3()
    position.setFromSpherical(spherical)
    return position
  }

  //精灵标签--标签永远面向相机
  function createTxt(position,name){
    let texture = new THREE.TextureLoader().load('');
    // let texture = new THREE.TextureLoader().load('/images/add-green.png');
    let material = new THREE.SpriteMaterial({ map: texture });
    let spriteMesh = new THREE.Sprite(material);

    /***设置mesh的name属性，用于鼠标点击标签弹出对应的名称***/
    spriteMesh.name = name;

    // 放大
    spriteMesh.scale.x = 1;
    spriteMesh.scale.y = 1;

    // spriteMesh.position.copy(position);	//原位置
    // 在原位置各加 1px ,避免图标与地球重叠
    spriteMesh.position.x = (position.x >0 ? position.x+1 : position.x-1);
    spriteMesh.position.y = (position.y >0 ? position.y+1 : position.y-1);
    spriteMesh.position.z = (position.z >0 ? position.z+1 : position.z-1);
    spriteMesh.lookAt(new THREE.Vector3(0, 0,0));
    scene.add(spriteMesh);
  }

  // 渲染
  function renderScene() {
    requestAnimationFrame(renderScene); //循环调用renderScene函数
    renderer.render(scene, camera);

    // 添加以下代码
    if(resizeDevicePixel(renderer)) {
      const canvas = renderer.domElement;
      camera.aspect = canvas.clientWidth / canvas.clientHeight;
      camera.updateProjectionMatrix();
    }
  }



}

// 导入模型
const gltfLoader = new GLTFLoader();
gltfLoader.load("/air_engine/scene.gltf", (gltf:any) => {
  let model = gltf.scene;

  // 设置模型的位置为原点
  model.position.set(0,0,0)

  // 设置模型的大小
  // model.scale.set(3.5, 3.5, 3.5);
  model.scale.set(model_scale, model_scale, model_scale)

  // 遍历模型
  // model.traverse((obj) => {
  //   // 将模型设置为线框模式
  //   obj.material = new THREE.MeshBasicMaterial({
  //     // wireframe: true,
  //     // color: 0xffffff,
  //     // // 设置线的宽度
  //     // wireframeLinewidth: 0.3,
  //     // // 设置线的透明度
  //     // transparent: true,
  //     // opacity: 0.2
  //   })
  // })

  scene.add(model);

  // 添加灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5); // 软光
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
  directionalLight["position"].set(1, 1, 1).normalize();

  scene.add(ambientLight);
  scene.add(directionalLight);

  // 设置点光源
  const pointLight = new THREE.PointLight(0xffffff, 1, 100);
  pointLight["position"].set(0, 50, 50);
  scene.add(pointLight);

  const light01 = new THREE.DirectionalLight(0xffffff, 1)
  light01["position"].set(0, 0, 10)
  const light02 = new THREE.DirectionalLight(0xffffff, 1)
  light02["position"].set(0, 0, -10)
  const light03 = new THREE.DirectionalLight(0xffffff, 1)
  light03["position"].set(10, 0, 0)
  const light04 = new THREE.DirectionalLight(0xffffff, 1)
  light04["position"].set(-10, 0, 0)
  const light05 = new THREE.DirectionalLight(0xffffff, 1)
  light05["position"].set(0, 10, 0)
  const light06 = new THREE.DirectionalLight(0xffffff, 1)
  light06["position"].set(5, 10, 0)
  const light07 = new THREE.DirectionalLight(0xffffff, 1)
  light07["position"].set(0, -10, 0)
  scene.add(light01)
  scene.add(light02)
  scene.add(light03)
  scene.add(light04)
  scene.add(light05)
  scene.add(light06)
  scene.add(light07)



})
scene.background = new THREE.Color(0xff0000);
// 所有元素一起旋转
setInterval(() => {
  scene["rotation"].y += speed.value
}, 1000 / 60)

onMounted(() => {
  initThree();
});


// 鼠标点击标记的事件
function clickMouse(event) {

  console.log("ddd")
  event.preventDefault();
  let raycaster = new THREE.Raycaster();
  let mouse = new THREE.Vector2();

  // 通过鼠标点击位置,计算出 raycaster 所点的位置,以屏幕为中心点,范围 -1 到 1
  let getBoundingClientRect = dom.getBoundingClientRect();
  mouse.x = ((event.clientX - getBoundingClientRect.left) / dom.offsetWidth) * 2 - 1;
  mouse.y = -((event.clientY - getBoundingClientRect.top) / dom.offsetHeight) * 2 + 1;

  //通过鼠标点击的位置(二维坐标)和当前相机的矩阵计算出射线位置
  raycaster.setFromCamera(mouse, camera);

  // 获取与射线相交的对象数组，其中的元素按照距离排序，越近的越靠前
  let intersects = raycaster.intersectObjects(scene.children);

  deleteDiv(); //调用清除弹窗函数

  // 获取点击对象的值
  for (let i = 0; i < intersects.length; i++) {
    if (intersects[i].object.type == 'Sprite') {
      let countryName = intersects[i].object.name; // 国家
      console.log("ming cheng",countryName)
      let V3 = intersects[i].object.position ; // 三维坐标
      if(countryName !=''){
        divPop(countryName, V3); //调用弹窗函数
      }
    }
  }


  // 弹窗内容与样式
  function divPop(countryName, V3){
    let rs = WorldToScreen(V3.x, V3.y, V3.z); // 调用世界坐标转屏幕坐标函数
    let div = document.createElement("divCell"); //创建一个div
    div.id = "divCell";  //设置ID
    div.innerHTML = countryName; //div的内容
    div.style.padding = '5px';
    div.style.position = 'absolute';
    div.style.backgroundColor = 'rgba(255, 255, 255, 0.8)';
    div.style.left = rs.x + "px";
    div.style.top = rs.y + "px";
    document.body.appendChild(div); //添加到页面
  }

  // 清除弹窗
  function deleteDiv(){
    //如果原来有“divCell”这个图层，先删除这个图层
    let d = document.getElementById("divCell");
    if (d != null){
      d.parentNode.removeChild(d);
    }
  }

  // 世界坐标转屏幕坐标
  function WorldToScreen(x, y, z) {
    let worldVector = new THREE.Vector3(x, y, z);
    let vector = worldVector.project(camera); //世界坐标转标准设备坐标
    let w = window.innerWidth / 2;
    let h = window.innerHeight / 2;
    return {
      x: Math.round(vector.x * w + w),
      y: Math.round(-vector.y * h + h)
    }
  }

}

// window.addEventListener('click', clickMouse, false);

function resizeDevicePixel(renderer) {
  const canvas = renderer.domElement
  let width = window.innerWidth
  let height = window.innerHeight
  let devicePixelWidth = canvas.width / window.devicePixelRatio
  let devicePixelHeight = canvas.height / window.devicePixelRatio

  const needResize = devicePixelWidth !== width || devicePixelHeight !== height
  if (needResize) {
    renderer.setSize(width, height, false)
  }
  return needResize
}

</script>

<template>
  <div class="centermap">
    <div class="maptitle">
      <div class="zuo"></div>
      <span class="titletext">{{ title }}</span>
      <div class="you"></div>
    </div>
    <div class="mapwrap">
      <BorderBox13>
        <canvas id="dom" ref="screenDom">
          <div id="tag">标签内容</div>
        </canvas>
      </BorderBox13>
    </div>
  </div>
</template>

<style scoped lang="scss">
// 以下为style常用变量
:modle_position {
  --model_positon_width: 760px;
  --model_position_height: 400px;
}

.centermap {
  margin-bottom: 30px;

  .maptitle {
    height: 60px;
    display: flex;
    justify-content: center;
    padding-top: 10px;
    box-sizing: border-box;

    .titletext {
      font-size: 28px;
      font-weight: 900;
      letter-spacing: 6px;
      background: linear-gradient(92deg, #0072ff 0%, #00eaff 48.8525390625%, #01aaff 100%);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin: 0 10px;
    }

    .zuo,
    .you {
      background-size: 100% 100%;
      width: 29px;
      height: 20px;
      margin-top: 8px;
    }

    .zuo {
      background: url("@/assets/img/xiezuo.png") no-repeat;
    }

    .you {
      background: url("@/assets/img/xieyou.png") no-repeat;
    }
  }

  .mapwrap {
    // 发动机窗口高度
    // height: 580px;
    height: 480px;
    // width: 500px;
    // padding: 0 0 10px 0;
    box-sizing: border-box;
    position: relative;

    .quanguo {
      position: absolute;
      right: 20px;
      top: -46px;
      width: 80px;
      height: 28px;
      border: 1px solid #00eded;
      border-radius: 10px;
      color: #00f7f6;
      text-align: center;
      line-height: 26px;
      letter-spacing: 6px;
      cursor: pointer;
      box-shadow: 0 2px 4px rgba(0, 237, 237, 0.5), 0 0 6px rgba(0, 237, 237, 0.4);
      z-index: 10;
    }
  }
}

#dom {
  // 模型的位置
  // width: 700px;
  width: 860px;
  // height: 580px;
  height: 470px;

  position: absolute;
  top: 0;
  left: 0;
}

</style>
