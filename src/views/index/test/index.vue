<script setup lang="ts">

import * as THREE from 'three';
import { onMounted } from "vue";
import { useSettingStore } from "@/stores";
import  { GLTFLoader } from "three/addons/loaders/GLTFLoader.js"; // gltf模型引入
import { OrbitControls } from "three/addons/controls/OrbitControls.js"; // 坐标轴控制器
import { CSS2DRenderer,CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js'; // CSS2标签
import { CSS3DRenderer,CSS3DObject } from 'three/addons/renderers/CSS3DRenderer.js';
import { ref, reactive, nextTick, watch } from "vue";

// 定义全局变量
// 创建一个场景，它将包含我们所有的元素，如物体，相机和灯光。

// 创建一个摄像机，它定义了我们正在看的地方



var radius = 100; // 地球半径
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

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    30,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
)


let threeDemo:any = ref(null);
function initThree() {
  scene.background = new THREE.Color("#eee");

  camera.position.z = 10;
  threeDemo = document.getElementById("dom");

  const renderer = new THREE.WebGLRenderer({canvas: threeDemo, antialias: true});

  // 环境光(颜色，亮度)
  var ambientLight = new THREE.AmbientLight("#ffffff", 1.2);
  scene.add(ambientLight);

  // 设置背景图
  var texture = new THREE.TextureLoader().load('images/back.jpg');
  scene.background = texture;


  // 显示坐标轴
  var axes = new THREE.AxesHelper(radius);
  scene.add(axes);

  // 添加轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update()

  // 将内容添加到HTML页面


  // 运行函数
  createEarth();
  createAreaPoint();
  renderScene();

  function annimate() {
    renderer.render(scene, camera);
    requestAnimationFrame(annimate);
  }
  annimate()

  // 创建地球 半径100
  function createEarth() {
    var earthGeo = new THREE.SphereGeometry(radius, 50, 50);
    earthGeo.name = '地球'
    var earthMater = new THREE.MeshPhongMaterial({
      map: new THREE.TextureLoader().load('/images/11.jpg'),
      //透明球体
      // transparent: true,
      // depthWrite: false,
      // side: THREE.DoubleSide,
      // blending: THREE.AdditiveBlending,
      // opacity: 0.8,
      // color: 0x03d98e
    });
    var earthMesh = new THREE.Mesh(earthGeo, earthMater);
    scene.add(earthMesh);
  }

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
  function createPosition(lnglat) {
    let spherical = new THREE.Spherical
    spherical.radius = radius;
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
    let texture = new THREE.TextureLoader().load('/images/22.jpg');
    let material = new THREE.SpriteMaterial({ map: texture });
    let spriteMesh = new THREE.Sprite(material);

    /***设置mesh的name属性，用于鼠标点击标签弹出对应的名称***/
    spriteMesh.name = name;

    // 放大
    spriteMesh.scale.x = 4;
    spriteMesh.scale.y = 4;

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
  }

};

// 鼠标点击标记的事件
function clickMouse(event) {
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


// const gltfLoader = new GLTFLoader();
// gltfLoader.load("/f6_boxer_engine/scene.gltf", (gltf) => {
//   let model = gltf.scene;
//
//   scene.add(model);
// })

// 随着窗体的变化修改场景大小
function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  console.log("调用了onRes方法")
}

// 监听窗体调整大小事件
window.addEventListener('resize', onResize, false);
// 监听鼠标事件
window.addEventListener('click', clickMouse, false);


onMounted(() => {

  initThree();
})


</script>

<template>
  <div>
    <canvas id="dom" style="width: 1500px;height: 1000px;border: 1px solid #000;"></canvas>
  </div>
</template>

<style scoped lang="scss">
body {
  margin: 0;
  overflow: hidden;
  border: 1px solid #000;
}
</style>
