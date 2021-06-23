<template>
    <div>
        <div id="container"></div>
        <div class="pad">
            <div v-if="camera">
                <div class="pad-title">Camera</div>
                <div>
                    <span class="pad-item-title">fov</span>
                    {{ camera.fov }}
                </div>
                <div>
                    <span class="pad-item-title">zoom</span>
                    {{ camera.zoom }}
                </div>
                <div>
                    <span class="pad-item-title">x</span>
                    {{ camera.position.x }}
                </div>
                <div>
                    <span class="pad-item-title">y</span>
                    {{ camera.position.y }}
                </div>
                <div>
                    <span class="pad-item-title">z</span>
                    {{ camera.position.z }}
                </div>

            </div>
        </div>
    </div>
</template>

<script>
    import * as THREE from 'three';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
    const plants = {};
    const track = {};
    const mixers = [];
    const clock = new THREE.Clock();
    const label = {};

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
                    radius: 10, // 436 1392 / 2 / 50
                    light: true,
                }).then(sun => {
                    this.createPlant('mercury', {
                        map: 'mercury.jpg',
                        radius: 4.82 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 57.90, 57.90, 2000)
                    });
                    this.createPlant('venus', {
                        map: 'venus.jpg',
                        radius: 12.3 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 108.2, 108.2, 4000)
                    });
                    this.createPlant('earth', {
                        map: 'earth.png',
                        radius: 12.6 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 149.6, 149.6, 20000)
                    }).then(earth => {
                        this.createPlant('moon', {
                            map: 'moon.jpg',
                            radius: 1.73,
                            track: this.createLineCircle(earth.position.x, earth.position.y, 38.4, 38.4, 1000)
                        });
                    });
                    this.createPlant('mars', {
                        map: 'mars.jpg',
                        radius: 6.72 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 227.9, 227.9, 25000)
                    });
                    this.createPlant('jupiter', {
                        map: 'jupiter.jpg',
                        radius: 142.3 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 778.3, 778.3, 28000)
                    }).then(jupiter => {
                        this.createPlant('jupiter-1', {
                            map: 'moon.jpg',
                            radius: 1.73,
                            track: this.createLineCircle(jupiter.position.x, jupiter.position.y, 160, 160, 3000)
                        });
                    });
                    this.createPlant('saturn', {
                        map: 'saturn.jpg',
                        radius: 123,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 1429.4,  1429.4, 30000)
                    });
                    this.createPlant('uranus', {
                        map: 'uranus.jpg',
                        radius: 53.3 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 2871, 2871, 26000)
                    });
                    this.createPlant('neptune', {
                        map: 'neptune.jpg',
                        radius: 49.2 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 4504, 4504, 22000)
                    });
                    this.createPlant('pluto', {
                        map: 'pluto.png',
                        radius: 2.2 / 2,
                        track: this.createLineCircle(sun.position.x, sun.position.y, 5000, 5000, 17000)
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

                                if(label[name]) {
                                    label[name].position.x = plants[name].position.x;
                                    label[name].position.y = plants[name].position.y + plants[name].geometry.parameters.radius;
                                    label[name].position.z = plants[name].position.z;
                                }
                            }
                        }
                        resolve(name);
                    }
                });
            },
            createScence() {
                return new Promise(resolve => {
                    this.camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 50, 8000 );
                    this.camera.position.x = 0;
                    this.camera.position.y = -300;
                    this.camera.position.z = 900;

                    this.scene = new THREE.Scene();

                    // 网格
                    // const size = 500;
                    // const divisions = 10;
                    // const gridHelper = new THREE.GridHelper( size, divisions, 'yellow', 'green'  );
                    // this.scene.add( gridHelper );

                    // 主光源
                    const light = new THREE.PointLight( 0xffffff, 1, 0, 1 );
                    light.position.set( 0, 0, 0 );
                    light.castShadow = false;
                    this.scene.add( light );

                    // 环境光
                    const sceneLight = new THREE.AmbientLight( 0xffffff, .2 ); // soft white light
                    this.scene.add( sceneLight );

                    // 平行光
                    const directionalLight = new THREE.DirectionalLight( 0xffffff, .2 );
                    directionalLight.position.set( -5000, 2, 0 );
                    directionalLight.position.normalize();
                    this.scene.add( directionalLight );

                    // 背景
                    const vertices = [];
                    for ( let i = 0; i < 2000; i ++ ) {
                        const x = THREE.MathUtils.randFloatSpread( 5000 );
                        const y = THREE.MathUtils.randFloatSpread( 5000 );
                        const z = THREE.MathUtils.randFloatSpread( 5000 );
                        vertices.push( x, y, z );
                    }
                    const geometry = new THREE.BufferGeometry();
                    geometry.setAttribute( 'position', new THREE.Float32BufferAttribute( vertices, 3 ) );
                    const material = new THREE.PointsMaterial({
                        color: 0x888888,
                        sizeAttenuation: false,
                    });
                    const points = new THREE.Points( geometry, material );
                    this.scene.add( points );

                    // control
                    this.renderer = new THREE.WebGLRenderer( { antialias: true } );
                    this.renderer.setClearColor(0x000019, 1);
                    this.renderer.shadowMap = true;
                    this.renderer.setSize( window.innerWidth, window.innerHeight );
                    this.renderer.setAnimationLoop( this.animation );

                    this.controls = new OrbitControls( this.camera, this.renderer.domElement );
                    this.controls.minDistance = 1;
                    this.controls.maxDistance = 2000;
                    document.getElementById('container').innerHTML = '';
                    document.getElementById('container').appendChild( this.renderer.domElement );

                    // this.createGlb();
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
                    color : 'rgb(50, 50, 50)',
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
                        this.createLabel(name);
                        resolve(plants[name]);
                    } );
                });

            },
            createLabel(name) {
                let canvas = document.createElement('canvas');
                let ctx = canvas.getContext('2d');
                ctx.fillStyle = "rgb(255, 255, 255)";
                ctx.font = "16px Arial";
                ctx.fillText(name.toUpperCase(), 130, 55);
                ctx.globalAlpha = 1;
                let texture = new THREE.Texture(canvas);
                texture.needsUpdate = true;
                let spriteMaterial = new THREE.SpriteMaterial({
                    map: texture,
                    sizeAttenuation: true,
                    transparent: true,
                    opacity: 1,
                });
                let sprite = new THREE.Sprite(spriteMaterial);
                sprite.position.set(0, 0, 0);
                sprite.scale.set(500, 200, 1);
                label[name] = sprite;
                this.scene.add(sprite);
            },
            createGlb() {
                const loader = new GLTFLoader();
                loader.setPath( './assets/' );
                loader.load( 'Parrot.glb', (gltf) => {
                    const mesh = gltf.scene.children[ 0 ];
                    const s = 1;
                    mesh.scale.set( s, s, s );
                    mesh.position.x = 100;
                    mesh.rotation.y = 100;
                    mesh.rotation.z = -100;
                    mesh.castShadow = true;
                    mesh.receiveShadow = true;
                    this.scene.add( mesh );
                    const mixer = new THREE.AnimationMixer( mesh );
                    mixer.clipAction( gltf.animations[ 0 ] ).setDuration( 1 ).play();
                    mixers.push( mixer );
                } );
            },
            animation(time) {
                this.createAimation('sun', time).then((sun) => {
                    this.createAimation('mercury', time, { follow: sun });
                    this.createAimation('venus', time, { follow: sun });
                    this.createAimation('earth', time, { follow: sun }).then((earth) => {
                        this.createAimation('moon', time, { follow: earth });
                    });
                    this.createAimation('mars', time, { follow: sun });
                    this.createAimation('jupiter', time, { follow: sun }).then((jupiter) => {
                        this.createAimation('jupiter-1', time, { follow: jupiter });
                    });
                    this.createAimation('saturn', time, { follow: sun });
                    this.createAimation('uranus', time, { follow: sun });
                    this.createAimation('neptune', time, { follow: sun });
                    this.createAimation('pluto', time, { follow: sun });
                });

                this.controls.update();
                const delta = clock.getDelta();
                for ( let i = 0; i < mixers.length; i ++ ) {
                    mixers[ i ].update( delta );
                }
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
    .pad {
        text-align: left;
        width: 300px;
        border: 1px solid #fff;
        color: #fff;
        position: fixed;
        top: 20px;
        left: 20px;
        padding: 8px;
        font-size: 10px;
        opacity: .5;
    }
    .pad-title {
        font-size: 14px;
        margin-bottom: 5px;
        text-transform: capitalize;
        font-weight: bold;
    }
    .pad-item-title {
        text-transform: capitalize;
    }
    .pad-item-title:after {
        content: ":";
    }
</style>
