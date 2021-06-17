<template>
    <div id="container"></div>
</template>

<script>
    import * as THREE from 'three';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    let camera, scene, renderer;
    let controls;
    let mesh;

    export default {
        name: 'App',
        components: {

        },
        mounted() {
            this.init();
        },
        methods: {
            init() {
                camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 500 );
                camera.position.x = 2;
                camera.position.y = 2;
                camera.position.z = 2;

                scene = new THREE.Scene();

                // 网格
                const size = 10;
                const divisions = 16;
                const gridHelper = new THREE.GridHelper( size, divisions, 'yellow', 'green'  );
                scene.add( gridHelper );

                // 点光源
                // 背景光
                const backLight = new THREE.PointLight( 0xffffff, 1, 20, 2 );
                backLight.position.set( -3, -3, -4 );
                backLight.castShadow = false;
                scene.add( backLight );

                // 主光源
                const light = new THREE.PointLight( 0xffffff, 1.4, 100, 2 );
                light.position.set( 2, 3, 4 );
                light.castShadow = false;
                scene.add( light );

                // 环境光
                const sceneLight = new THREE.AmbientLight( 0x111111, 1 ); // soft white light
                scene.add( sceneLight );

                // 平行光
                const directionalLight = new THREE.DirectionalLight( 0xffffff, .2 );
                directionalLight.position.set( -5, 2, 0 );
                directionalLight.position.normalize();
                scene.add( directionalLight );

                // 模型
                const geometry = new THREE.BoxGeometry( 1, 1, 1 );

                // 材质
                const loader = new THREE.TextureLoader();
                loader.setPath( '../assets/' );

                loader.load( 'box.jpeg', ( texture ) => {
                    const material = new THREE.MeshPhongMaterial( {
                        color: 0xffffff,
                        side: THREE.FrontSide,
                        map: texture,
                    });

                    mesh = new THREE.Mesh( geometry, material );
                    mesh.position.y = .5;
                    mesh.matrixAutoUpdate = true;
                    mesh.receiveShadow = true;
                    scene.add(mesh);

                    renderer = new THREE.WebGLRenderer( { antialias: true } );
                    renderer.shadowMap = true;
                    renderer.setSize( window.innerWidth, window.innerHeight );
                    renderer.setAnimationLoop( this.animation );

                    controls = new OrbitControls( camera, renderer.domElement );
                    controls.minDistance = 1;
                    controls.maxDistance = 500;

                    document.body.innerHTML = '';
                    document.body.appendChild( renderer.domElement );
                } );

            },
            animation() {
                // mesh.rotation.x += 0.01;
                // mesh.rotation.y += 0.01;
                // mesh.rotation.z += 0.01;
                renderer.render( scene, camera );
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
