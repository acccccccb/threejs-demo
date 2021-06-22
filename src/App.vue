<template>
    <div>
        <div id="container"></div>
        <div id="vertexShader"></div>
        <div id="fragmentShader"></div>
    </div>
</template>

<script>
    import * as THREE from 'three';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    const plants = {};
    const track = {};

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
            this.createScence().then(() => {
                this.createPlant('sun', {
                    map: 'sun.jpg',
                    radius: 10, // 436
                    light: true,
                }).then(sun => {
                    this.createPlant('mercury', {
                        map: 'venus.jpg',
                        radius: 1.2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 30, 30, 2000)
                    });
                    this.createPlant('venus', {
                        map: 'venus.jpg',
                        radius: 1.5,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 50, 50, 4000)
                    });
                    this.createPlant('earth', {
                        map: 'earth.png',
                        radius: 4,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 80, 80, 20000)
                    }).then(earth => {
                        this.createPlant('moon', {
                            map: 'moon.jpg',
                            radius: 1,
                            track: this.createLineCircle(earth.position.x, earth.position.y, 10, 10, 1000)
                        });
                    });
                    this.createPlant('mars', {
                        map: 'mars.jpg',
                        radius: 4.5,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 110, 110, 25000)
                    });
                    this.createPlant('jupiter', {
                        map: 'jupiter.jpg',
                        radius: 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 130, 130, 28000)
                    });
                    this.createPlant('saturn', {
                        map: 'venus.jpg',
                        radius: 4,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 150, 150, 30000)
                    });
                    this.createPlant('uranus', {
                        map: 'venus.jpg',
                        radius: 3,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 180, 180, 26000)
                    });
                    this.createPlant('neptune', {
                        map: 'venus.jpg',
                        radius: 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 200, 200, 22000)
                    });
                    this.createPlant('pluto', {
                        map: 'venus.jpg',
                        radius: 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 230, 230, 17000)
                    });
                });

            });
        },
        methods: {
            createAimation(name, time, params) {
                return new Promise(resolve => {
                    if(plants[name]) {
                        plants[name].rotation.y += 0.01;
                        if(track[name]) {
                            const points = track[name];
                            const deviationX = plants[params.follow] ? plants[params.follow].position.x : 0;
                            const deviationY = plants[params.follow] ? plants[params.follow].position.y : 0;
                            if(plants[name] && plants[name].position) {
                                plants[name].position.x =
                                    points[((time) % points.length).toFixed(0)].x + deviationX;
                                plants[name].position.y =
                                    points[((time) % points.length).toFixed(0)].y + deviationY;
                            }
                        }
                        resolve(name);
                    }
                });
            },
            createScence() {
                return new Promise(resolve => {
                    this.camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 50, 5000 );
                    this.camera.position.x = 0;
                    this.camera.position.y = 5;
                    this.camera.position.z = 500;

                    this.scene = new THREE.Scene();

                    // 网格
                    // const size = 500;
                    // const divisions = 10;
                    // const gridHelper = new THREE.GridHelper( size, divisions, 'yellow', 'green'  );
                    // this.scene.add( gridHelper );

                    // 点光源
                    // 背景光
                    // const backLight = new THREE.PointLight( 0xffffff, .2, 20, 2 );
                    // backLight.position.set( -3, -3, -4 );
                    // backLight.castShadow = false;
                    // this.scene.add( backLight );

                    // 主光源
                    const light = new THREE.PointLight( 0xffffff, 2, 500, 1 );
                    light.position.set( 0, 0, 0 );
                    light.castShadow = false;
                    this.scene.add( light );

                    // 环境光
                    const sceneLight = new THREE.AmbientLight( 0xffffff, .1 ); // soft white light
                    this.scene.add( sceneLight );

                    // 平行光
                    const directionalLight = new THREE.DirectionalLight( 0xffffff, .2 );
                    directionalLight.position.set( -5, 2, 0 );
                    directionalLight.position.normalize();
                    this.scene.add( directionalLight );

                    // 背景
                    const vertices = [];
                    for ( let i = 0; i < 5000; i ++ ) {
                        const x = THREE.MathUtils.randFloatSpread( 2000 );
                        const y = THREE.MathUtils.randFloatSpread( 2000 );
                        const z = THREE.MathUtils.randFloatSpread( 2000 );
                        vertices.push( x, y, z );
                    }
                    const geometry = new THREE.BufferGeometry();
                    geometry.setAttribute( 'position', new THREE.Float32BufferAttribute( vertices, 3 ) );
                    const material = new THREE.PointsMaterial( { color: 0x888888 } );
                    const points = new THREE.Points( geometry, material );
                    this.scene.add( points );

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
            createLineCircle(x = 0, y = 0, radiusX, radiusY, speed = 1, clockwise = true) {
                const curve = new THREE.EllipseCurve(
                    x,  y,            // ax, aY
                    radiusX, radiusY,           // xRadius, yRadius
                    0,  2 * Math.PI,  // aStartAngle, aEndAngle
                    clockwise,            // aClockwise
                    90,                 // aRotation
                );
                const points = curve.getPoints( (speed - 1) || 0 );

                const geometry = new THREE.BufferGeometry().setFromPoints( points );
                const material = new THREE.LineBasicMaterial( {
                    color : 'rgb(20, 20, 20)',
                    linewidth: 1,
                } );
                const ellipse = new THREE.Line( geometry, material );
                this.scene.add( ellipse );

                return points;
            },
            createPlant(name, params) {
                return new Promise(resolve => {
                    const geometry = new THREE.SphereGeometry( params.radius, 128, 128 );
                    // 材质
                    const loader = new THREE.TextureLoader();
                    loader.setPath( './assets/' );

                    loader.load( params.map, ( texture ) => {
                        let material;
                        if(params.light) {
                            material = new THREE.MeshStandardMaterial({
                                side: THREE.FrontSide,
                                color: 0xffffff,
                                map: texture,
                                emissive: 0xffffff,
                                emissiveMap: texture,
                            });
                        } else {
                            material = new THREE.MeshStandardMaterial({
                                side: THREE.FrontSide,
                                color: 0xffffff,
                                map: texture,
                            });
                        }

                        plants[name] = new THREE.Mesh( geometry, material );

                        plants[name].matrixAutoUpdate = true;
                        plants[name].receiveShadow = true;
                        this.scene.add(plants[name]);
                        if(params.track) {
                            track[name] = params.track;
                        }
                        resolve(plants[name]);
                    } );
                });

            },
            animation(time) {
                this.createAimation('sun', time).then((sun) => {
                    this.createAimation('mercury', time, { follow: sun });
                    this.createAimation('venus', time, { follow: sun });
                    this.createAimation('earth', time, { follow: sun }).then((earth) => {
                        this.createAimation('moon', time, { follow: earth });
                    });
                    this.createAimation('mars', time, { follow: sun });
                    this.createAimation('jupiter', time, { follow: sun });
                    this.createAimation('saturn', time, { follow: sun });
                    this.createAimation('neptune', time, { follow: sun });
                    this.createAimation('pluto', time, { follow: sun });
                });

                this.controls.update();
                this.renderer.render( this.scene, this.camera );
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
