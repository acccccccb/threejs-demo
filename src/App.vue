<template>
    <div>
        <div id="container"></div>
    </div>
</template>

<script>
    import * as THREE from 'three';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    let moon, earth;

    export default {
        name: 'App',
        data() {
            return {
                camera: null,
                scence: null,
                renderer: null,
                controls: null,
                earth: null,
                moon: null,
                start: 0
            }
        },
        mounted() {
            // this.init();
            this.createScence().then(() => {
                this.createEarth();
                this.createMoon();
                this.createLineCircle();
            });
        },
        methods: {
            animation(time) {
                if(earth) {
                    earth.rotation.y += 0.01;
                }
                if(moon) {
                    const points = this.createLineCircle(0, 0, 330);
                    moon.position.x = points[((time)%points.length).toFixed(0)].x;
                    moon.position.y = points[((time)%points.length).toFixed(0)].y;
                }
                this.controls.update();
                this.renderer.render( this.scene, this.camera );
            },
            createScence() {
                return new Promise(resolve => {
                    this.camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 5000 );
                    this.camera.position.x = 0;
                    this.camera.position.y = 5;
                    this.camera.position.z = 800;

                    this.scene = new THREE.Scene();

                    // 网格
                    // const size = 500;
                    // const divisions = 100;
                    // const gridHelper = new THREE.GridHelper( size, divisions, 'yellow', 'green'  );
                    // this.scene.add( gridHelper );

                    // 点光源
                    // 背景光
                    const backLight = new THREE.PointLight( 0xffffff, .2, 20, 2 );
                    backLight.position.set( -3, -3, -4 );
                    backLight.castShadow = false;
                    this.scene.add( backLight );

                    // 主光源
                    const light = new THREE.PointLight( 0xffffff, 1, 100, 2 );
                    light.position.set( 2, 3, 4 );
                    light.castShadow = false;
                    this.scene.add( light );

                    // 环境光
                    const sceneLight = new THREE.AmbientLight( 0xffffff, 2 ); // soft white light
                    this.scene.add( sceneLight );

                    // 平行光
                    const directionalLight = new THREE.DirectionalLight( 0xffffff, .2 );
                    directionalLight.position.set( -5, 2, 0 );
                    directionalLight.position.normalize();
                    this.scene.add( directionalLight );

                    // control
                    this.renderer = new THREE.WebGLRenderer( { antialias: true } );
                    this.renderer.setClearColor(0x000000, 1);
                    this.renderer.shadowMap = true;
                    this.renderer.setSize( window.innerWidth, window.innerHeight );
                    this.renderer.setAnimationLoop( this.animation );

                    this.controls = new OrbitControls( this.camera, this.renderer.domElement );
                    this.controls.minDistance = 1;
                    this.controls.maxDistance = 2000;
                    document.body.innerHTML = '';
                    document.body.appendChild( this.renderer.domElement );
                    resolve();
                });
            },
            createEarth() {
                const geometry = new THREE.SphereGeometry( 11, 128, 128 );
                // 材质
                const loader = new THREE.TextureLoader();
                loader.setPath( './assets/' );

                loader.load( 'earth.jpg', ( texture ) => {
                    const material = new THREE.MeshStandardMaterial( {
                        color: 0xffffff,
                        side: THREE.FrontSide,
                        map: texture,
                    });

                    earth = new THREE.Mesh( geometry, material );
                    earth.position.x = 0;
                    earth.position.y = 0;
                    earth.matrixAutoUpdate = true;
                    earth.receiveShadow = true;
                    this.scene.add(earth);
                } );
            },
            createMoon() {
                const geometry = new THREE.SphereGeometry( 3, 128, 128 );
                // 材质
                const loader = new THREE.TextureLoader();
                loader.setPath( './assets/' );

                loader.load( 'moon.jpg', ( texture ) => {
                    const material = new THREE.MeshStandardMaterial( {
                        color: 0xffffff,
                        side: THREE.FrontSide,
                        map: texture,
                    });

                    moon = new THREE.Mesh( geometry, material );
                    moon.position.x = -330;
                    moon.position.y = 0;
                    moon.matrixAutoUpdate = true;
                    moon.receiveShadow = true;
                    this.scene.add(moon);
                } );
            },
            createLineCircle(x, y, radius) {
                const curve = new THREE.EllipseCurve(
                    x,  y,            // ax, aY
                    radius, radius,           // xRadius, yRadius
                    0,  2 * Math.PI,  // aStartAngle, aEndAngle
                    true,            // aClockwise
                    90,                 // aRotation
                );
                const points = curve.getPoints( 20000 );

                // const geometry = new THREE.BufferGeometry().setFromPoints( points );
                // const material = new THREE.LineBasicMaterial( {
                //     color : 0xdedede,
                //     linewidth: 1,
                // } );
                // const ellipse = new THREE.Line( geometry, material );
                // this.scene.add( ellipse );

                return points;
            },
        }
    }
</script>

<style>
    html, body {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
        overflow-x: hidden;
    }
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin: 0;
        padding: 0;
    }
</style>
