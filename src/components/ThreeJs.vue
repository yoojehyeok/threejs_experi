<template>
<div>
    <!-- <canvas id="canvas" ref="canvas"></canvas> -->
    <div ref="container" id="container">

    </div>
    <!-- <img src="/char.png"/> -->
</div>
</template>
<script>
import * as THREE from 'three'

import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { Line2 } from 'three/addons/lines/Line2.js';
import { LineMaterial } from 'three/addons/lines/LineMaterial.js';
import { LineGeometry } from 'three/addons/lines/LineGeometry.js';


    export default {
        data(){
            return{
                scene : {},
                camera : {},
                cameraPerspectiveHelper : {},
                renderer : {},
                controls : {},
                drawLogoGeometry : {},
                material : {},
                cube : {},
                backgroundMaterial : {},
                line : {},
                runtime : 5,
                zoom : 100,
                drawLogoMaterial : {},

                draw_x : 0,
                draw_y : 0,
                draw_z : 0,

                currentPosition : 0,

                logoPoints : [],
                logoPointsTotal : [],
                logoPositionArray : [


                    {x : 0, y : 0, z : 0, jumpPosition : true},
                    {x : 10, y : 10, z : 0, jumpPosition : false},
                    {x : 10, y : -10, z : 0, jumpPosition : false},
                    {x : 0, y : -20, z : 0, jumpPosition : false},
                    {x : -10, y : -10, z : 0, jumpPosition : false},
                    {x : -10, y : 10, z : 0, jumpPosition : false},
                    {x : 0, y : 20, z : 0, jumpPosition : false,},
                    {x : 0, y : -20, z : 0, jumpPosition : false},

                ],

                color : {},
                colors : [],

                animateId : null,
                isCloseUpAnimationFlag : false,
                playAnimation : ["animate", "closeUpAnimate"],
                currentAnimate : 0,
                zoomInId : {},

                endLogoTexture : {},
                lineArray : [],
                roomMesh : {},
                direction : "up",
                
            }
        },
        mounted(){
            const _this = this;
            this.renderer = new THREE.WebGLRenderer({antialias : true});
            this.renderer.setClearColor( 0x000000, 0.0 );
            this.renderer.setSize( window.innerWidth, window.innerHeight );
            this.$refs.container.appendChild( this.renderer.domElement );


            this.scene = new THREE.Scene();
            this.camera = new THREE.PerspectiveCamera( 40, window.innerWidth / window.innerHeight, 1, 1000 );
            // this.camera.position.set( - 40, 0, 60 );
            // this.cameraPerspectiveHelper = new THREE.CameraHelper( this.camera );
            // this.scene.add(this.cameraPerspectiveHelper);
            this.camera.position.set(0,0,this.zoom);
            // this.camera.zoom = 100;

            this.camera.lookAt(0,0,0);

            this.controls = new OrbitControls(this.camera, this.renderer.domElement);

            this.controls.rotateSpeed = 1.0; // 마우스로 카메라를 회전시킬 속도입니다. 기본값(Float)은 1입니다.
            this.controls.zoomSpeed = 1.2; // 마우스 휠로 카메라를 줌 시키는 속도 입니다. 기본값(Float)은 1입니다.
            this.controls.panSpeed = 0.8; // 패닝 속도 입니다. 기본값(Float)은 1입니다.
            this.controls.minDistance = 5; // 마우스 휠로 카메라 거리 조작시 최소 값. 기본값(Float)은 0 입니다.
            this.controls.maxDistance = 100; // 마우스 휠로 카메라 거리 조작시 최대 값. 기본값(Float)은 무제한 입니다.
            this.controls.maxPolarAngle = Math.PI / 2.1;    // 카메라 하단 각도
            this.controls.minPolarAngle = Math.PI / 10; // 카메라 상단 각도


            var imageLoader = new THREE.TextureLoader();
            imageLoader.load("/mesh.jpeg", function(data){
                _this.backgroundMaterial = new THREE.MeshBasicMaterial({
                    map : data,
                    side : THREE.BackSide
                })

                var backgroundGeometry = new THREE.SphereGeometry(300,32,32);
                _this.roomMesh = new THREE.Mesh(backgroundGeometry,_this.backgroundMaterial);
                _this.roomMesh.position.set(0,0,0);
                _this.scene.add(_this.roomMesh);
            })

            this.controls.update(); 
            
            this.drawLogoMaterial = new LineMaterial( {
                color: 0xffffff,
                linewidth: 5, // in world units with size attenuation, pixels otherwise
                vertexColors: true,
            } );

            this.initCurrentPoint(this.logoPositionArray);
            this.animate();
        },
        methods : {

            init(){
                this.draw_x = 0;
                this.draw_y = 0;
                this.draw_z = 0;
                this.currentPosition = 0;
                this.runtime = 5;
            },

            initCurrentPoint(pointArray){
                this.draw_x = pointArray[this.currentPosition].x;
                this.draw_y = pointArray[this.currentPosition].y;
                this.draw_z = pointArray[this.currentPosition].z;
            },

            pushLogoPoint(points){
                
                points.push( new THREE.Vector3( this.draw_x,
                                                this.draw_y,
                                                this.draw_z 
                                            )
                    );
                this.positions = [];
                this.color = new THREE.Color();
                points.map((e) => {
                    this.positions.push( e.x, e.y, e.z );
                    this.color.setHSL( this.runtime/300, 1.0, 0.5);
                    this.colors.push(this.color.r, this.color.g, this.color.b)
                })
                // console.log(this.runtime / 786)
                // console.log(this.positions);
            },
            changeLogoPoint(pointArray){
                
                if((pointArray[this.currentPosition+1].x - this.draw_x ) > 0){
                    this.draw_x += 0.5;
                }else if((pointArray[this.currentPosition+1].x - this.draw_x ) < 0){
                    this.draw_x -= 0.5;
                }
                if((pointArray[this.currentPosition+1].y - this.draw_y ) > 0){
                    this.draw_y += 0.5;
                }else if((pointArray[this.currentPosition+1].y - this.draw_y ) < 0){
                    this.draw_y -= 0.5;
                }
                if((pointArray[this.currentPosition+1].z - this.draw_z ) > 0){
                    this.draw_z += 0.5;
                }else if((pointArray[this.currentPosition+1].z - this.draw_z ) < 0){
                    this.draw_z -= 0.5;
                }

                this.drawLogoGeometry = new LineGeometry();
                this.drawLogoGeometry.setPositions(this.positions);
                this.drawLogoGeometry.setColors(this.colors);

                this.line = new Line2(this.drawLogoGeometry, this.drawLogoMaterial);
                this.lineArray.push(this.line);
                this.line.scale.set( 1, 1, 1 );
                this.scene.add(this.line);

            },
            compareLogoPoint(pointArray){
                if( this.draw_x == pointArray[this.currentPosition+1].x &&
                    this.draw_y == pointArray[this.currentPosition+1].y &&
                    this.draw_z == pointArray[this.currentPosition+1].z 
                ){
                    this.currentPosition +=1;
                    this.isEndPosition(pointArray);
                }else{
                    this.isRender = true;
                    this.changeLogoPoint(pointArray);
                }
            },
            drawLogo(pointArray, points){
                this.pushLogoPoint(points);
                this.compareLogoPoint(pointArray);
            },
            isEndPosition(pointArray){
                const _this = this;
                let lineWidthId = {};
                if(this.currentPosition == pointArray.length-1){
                    cancelAnimationFrame(this.animateId);
                    
                    lineWidthId = setInterval(function()
                    {
                        console.log(_this.drawLogoMaterial);
                        _this.drawLogoMaterial.linewidth += 0.2;


                        // _this.line.scale.set( 1, 1, 1 );
                        // _this.scene.add(_this.line);
                        _this.renderer.render( _this.scene, _this.camera );
                    
                    }, 10 );
                    setTimeout( () => {
                        clearInterval(lineWidthId);
                        this.endZoom();
                        this.render();
                    }, 1000);
                }
            },
            endZoom(){
                const _this = this;
                this.zoomInId = setInterval(function()
                {
                    _this.camera.zoom += _this.runtime/50000;
                    _this.camera.updateProjectionMatrix();
                }, 10 );

                setTimeout( () => {
                    clearInterval(this.zoomInId);
                    this.endLogo();
                }, 3000);
                
            },
            endLogo(){
                this.lineArray.map((o) => {
                    this.scene.remove(o);
                })
                this.drawLogoGeometry.dispose();
                this.drawLogoMaterial.dispose();
                // this.drawLogoMaterial.dispose();
                // this.line.dispose();


                var _this = this;
                // var endLogoTexture = new THREE.TextureLoader().load('/char.png');
                var endLogoTexture = new THREE.TextureLoader().load('/endLogo.png');
                var material = new THREE.SpriteMaterial( { map : endLogoTexture, color: 0xffffff });
                var sprite = new THREE.Sprite(material);
                sprite.scale.set(50,10,1);
                this.scene.add(sprite);

                this.zoomInId = setInterval(function()
                {
                    _this.camera.zoom -= _this.runtime/50000;
                    _this.camera.updateProjectionMatrix();
                }, 10 );

                setTimeout( () => {
                    clearInterval(this.zoomInId);
                }, 3000);
            },
            animate() {
                // this.camera.zoom += this.runtime/100000;
                // console.log(this.camera.zoom);
                // this.camera.updateProjectionMatrix();
                // console.log(this.camera);

                const _this = this;
                this.animateId = requestAnimationFrame(this.animate);

                this.drawLogoMaterial.resolution.set( window.innerWidth, window.innerHeight ); // resolution of the viewport

                if(_this.runtime % 2 == 0){
                    this.drawLogo(this.logoPositionArray, this.logoPoints);
                    
                    // this.drawLogoGeometry = new LineGeometry();
                    // this.drawLogoGeometry.setPositions(this.logoPointsTotal);
                    // this.drawLogoGeometry.setColors(this.colors);

                    // this.line = new Line2(this.drawLogoGeometry, this.drawLogoMaterial);

                    // this.line.scale.set( 1, 1, 1 );
                    // this.scene.add(this.line);
                    this.renderer.render( this.scene, this.camera );
                }
                this.runtime = this.runtime +1;
            },
            render(){

                requestAnimationFrame(this.render);
                if(this.roomMesh.position.y > 150){
                    this.direction = "up"
                }else if(this.roomMesh.position.y < -150){
                    this.direction = "down"
                }
                if(this.direction == "up"){
                    this.roomMesh.position.y -= 0.1;
                }else{
                    this.roomMesh.position.y += 0.1;
                }
                this.renderer.render( this.scene, this.camera );
            }

        }
}
</script>

<style>
    body { margin: 0; }
    canvas { display: block; }

</style>
