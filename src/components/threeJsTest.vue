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
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";

const $ = s => document.querySelector(s);
//展示模型
let showModel = null;
let showModelBone = null;
//摄像头
let camera = null;
//场景
let scene = null;
//动画
let animationMixers = [];
let clock = new THREE.Clock();
//动画文件路径
let animationPath = 'static/model/Naria@idie.FBX';
//灯光
let light = null;
//渲染器
let render = null;
//用户交互插件
let controls = null;
//模型路径
let modelPath = 'static/model/Naria.FBX';
//模型材质
let modelMat = null;
//shader路径
let shaderPath = 'static/shader/ChacterBodyShader';
//shader
let fragShaderStr = null;
let vertexShaderStr = null;
export default {
    name: 'threeJsTest',
    data() {
        return {

        };
    },
    methods: {
        //本地文件读取
        load(name) {
            let xhr = new XMLHttpRequest(),
                okStatus = document.location.protocol === "file:" ? 0 : 200;
            xhr.open('GET', name, false);
            xhr.overrideMimeType("text/html;charset=utf-8");//默认为utf-8
            xhr.send(null);
            return xhr.status === okStatus ? xhr.responseText : null;
        },

        //材质初始化
        initMat() {
            fragShaderStr = this.load(shaderPath + `.frag`);
            vertexShaderStr = this.load(shaderPath + `.vert`);

            modelMat = new THREE.ShaderMaterial({
                uniforms: {
                    _mainColor: { value: new THREE.Vector3(1.0, 1.0, 1.0) },
                    lightPosition: { value: new THREE.Vector3(0, -10, 0) },
                    tilling: { value: new THREE.Vector2(1, 1) },
                    _normalTex: { value: new THREE.TextureLoader().load("static/texture/Naria/Naria_N.tga") },
                    _roughness: { value: 1.0 },
                    _roughnessContrast: { value: 1.06 },
                    _roughnessInit: { value: 1.92 },
                    _roughnessMin: { value: 0.0 },
                    _roughnessMax: { value: 0.7 }
                },
                //236,65,65
                vertexShader: vertexShaderStr,
                fragmentShader: fragShaderStr,


            });
        },

        //场景初始化
        initScene() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xa0a0a0);
        },
        //初始化摄像头
        initCamera() {
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
            // -7.951022465039643, y: 50.66599857875026, z: 84.02389415272548
            camera.position.set(-7.951022465039643, 50.66599857875026, 84.02389415272548);
            console.log(camera);

            camera.lookAt(new THREE.Vector3(0, 0, 0));
        },
        //初始化灯光
        initLight() {
            scene.add(new THREE.AmbientLight(0x444444));
            light = new THREE.PointLight(0xffffff);
            // light.position.set(0, 1.25, 1.25);
            light.position.set(0, -110, 20);
            //告诉点光需要开启阴影投射
            light.castShadow = true;
            scene.add(light);
            let Ambient = new THREE.AmbientLight(0x404040, 2);
            scene.add(Ambient);
        },
        //模型初始化
        initModelFbx() {
            console.log('模型加载');

            let loader = new FBXLoader();

            loader.load(modelPath, function (object) {
                //创建纹理
                var mat = modelMat;
                console.log(object);
                showModel = object;
                showModel.position.set(0, -30, 0);
                // geometry.center(); //居中显示
                showModel.children[1].material =  mat;
                //获取原生shader代码
                let fragStr=THREE.ShaderLib["basic"].fragmentShader;
                let VertStr=THREE.ShaderLib["basic"].vertexShader;
                console.log(fragStr);
                console.log(VertStr);
                // showModel.children[1].material=mat;
                
                //添加骨骼辅助
                // let meshHelper = new THREE.SkeletonHelper(showModel);
                // scene.add(meshHelper);
                scene.add(showModel);
                console.log(showModel);
                console.log(showModel.children[1]);
            });

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
            controls = new OrbitControls(camera, render.domElement);
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
        //载入模型动画
        initModelAnim() {
            console.log('动画加载');

            let loader = new FBXLoader();

            loader.load(animationPath, function (object) {

                //创建动画混合器，并指定模型，混合器会自动根据指定模型寻找骨骼，并绑定
                let mixer = new THREE.AnimationMixer(showModel);
                //添加至动画混合器数组
                animationMixers.push(mixer);
                //挂载动画
                showModel.animations.push(object.animations[0]);
                //获取动画片
                let action = mixer.clipAction(showModel.animations[0]);
                //播放动画片
                action.play();

               

            });
        },

        render() {
            render.render(scene, camera);


        },

        //窗口变动触发的函数
        onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            this.render();
            render.setSize(window.innerWidth, window.innerHeight);
        },

        animate() {
            //更新
            requestAnimationFrame(this.animate);
            
            if (animationMixers.length > 0) {


                //遍历并更新所有动画混合器
                animationMixers[0].update(clock.getDelta());



            }
           
            this.render();


        },
        //绘制
        draw() {
            this.initScene();
            this.initMat();

            this.initCamera();
            this.initRender();
            this.initLight();
            this.initModelFbx();
            this.initControls();
            this.initModelAnim();
            this.animate();
            window.onresize = this.onWindowResize;
        }

    },
    mounted() {
        this.draw();
    }
}
</script>
