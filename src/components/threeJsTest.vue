<template>
    <div id="container">

    </div>
</template>
<style scoped>
#container {
    width: 100vw;
    height: 100vh;
}
</style>
<script>
import * as THREE from 'three';
const $ = s => document.querySelector(s);
//展示模型
let showModel = null;
//摄像头
let camera = null;
//场景
let scene = null;
//动画
let animationMixer = new THREE.animationMixer();
//灯光
let light = null;
//渲染器
let render = null;
//用户交互插件
let controls=null;
//模型路径
let modelPath='../model/Naria.FBX';
//模型材质
let modelMat=null;
//shader路径
export default {
    name: 'threeJsTest',
    data() {
        return {

        };
    },
    methods: {
        //材质初始化
        initMat() {
           
        },
        //本地文件读取
        load(name) {
            let xhr = new XMLHttpRequest(),
                okStatus = document.location.protocol === "file:" ? 0 : 200;
            xhr.open('GET', name, false);
            xhr.overrideMimeType("text/html;charset=utf-8");//默认为utf-8
            xhr.send(null);
            return xhr.status === okStatus ? xhr.responseText : null;
        },
        //场景初始化
        initScene() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xa0a0a0);
        },
        //初始化摄像头
        initCamera() {
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(31.10476063070969, 5.39751544957362, -197.53318883944013);
            camera.lookAt(new THREE.Vector3(0, 0, 0));
        },
        //初始化灯光
        initLight() {
            scene.add(new THREE.AmbientLight(0x444444));
            light = new THREE.PointLight(0xffffff);
            light.position.set(0, 1.25, 1.25);
            //告诉点光需要开启阴影投射
            light.castShadow = true;
            scene.add(light);
        },
        //渲染器初始化
        initRender() {
            render = new THREE.WebGLRenderer({ antialias: true });
            render.setSize(window.innerWidth, window.innerHeight);
            //修改渲染器输出格式
            render.outputEncoding = THREE.sRGBEncoding;
            //渲染器添加toneMapping效果
            // render.toneMapping = THREE.ACESFilmicToneMapping;
            //告诉渲染器需要阴影效果 
            render.setClearColor('#1F2025', 1.0);
            $('#container').appendChild(render.domElement);
        },
        //用户插件初始化
        initControls() {
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            // 使动画循环使用时阻尼或自转 意思是否有惯性
            controls.enableDamping = true;
            //动态阻尼系数 就是鼠标拖拽旋转灵敏度
            //controls.dampingFactor = 0.25;
            //是否可以缩放
            controls.enableZoom = true;
            //是否自动旋转
            controls.autoRotate = true;
            controls.autoRotateSpeed = 0.5;
            //设置相机距离原点的最远距离
            controls.minDistance = 1;
            //设置相机距离原点的最远距离
            controls.maxDistance = 500;
            //是否开启右键拖拽
            controls.enablePan = true;
        },
        //模型动画初始化
        initModelAnim()
        {

        },
        render() {
            render.render(scene, camera);

        },

        //窗口变动触发的函数
        onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            render();
            render.setSize(window.innerWidth, window.innerHeight);
        },

        animate() {
            //更新
            render();
            requestAnimationFrame(animate);
        }

    }
}
</script>
