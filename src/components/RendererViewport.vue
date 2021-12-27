<template>
    <div class="RendererViewport" ref="rendererTarget" />
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
    props: {
        pointCoordinates: Array,
        currentFrameColors: Array,
        cameraPosition: Array,
        pointStrength: {
            type: Number,
            default: 1,
        },
        pointSize: {
            type: Number,
            default: 0.5
        },
        maskTexture: {
            type: String,
            default: "/sphereMask_default.png",
        }
    },
    watch: {
        pointCoordinates() {
            this.updatePositions();
        },
        currentFrameColors() {
            this.updateColors();
        },
        cameraPosition() {
            this.updateCamera();
        },
        pointSize() {
            this.material.size = this.pointSize;
        },
        pointStrength() {
            this.material.color = new THREE.Color(
                this.pointStrength,
                this.pointStrength,
                this.pointStrength,
            );
        },
        maskTexture() {
            this.material.map = new THREE.TextureLoader().load(this.maskTexture);
        }
    },
    data() {
        return {
            camera: null,
            controls: null,
            scene: null,
            geometry: null,
            material: null,
            renderer: null,
            points: null,
            frameRequest: null,
        };
    },
    mounted() {
        const el = this.$refs.rendererTarget;

        if(this.camera !== null) throw 'Unexpected state';
        this.camera = new THREE.PerspectiveCamera(60, el.clientWidth / el.clientHeight, 0.01, 50);
        this.camera.up = new THREE.Vector3(0, 0, 1);

        this.scene = new THREE.Scene();
        this.scene.background = new THREE.Color(0x050505);
        this.scene.fog = new THREE.Fog(0x050505, 20, 50);

        this.geometry = new THREE.BufferGeometry();
        
        this.material = new THREE.PointsMaterial({ 
            size: this.pointSize, 
            vertexColors: true,
            map: new THREE.TextureLoader().load(this.maskTexture), 
            blending: THREE.AdditiveBlending,
            depthTest: false,
        });
        this.points = new THREE.Points(this.geometry, this.material);
        this.points.frustumCulled = false;
        this.scene.add(this.points);

        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setPixelRatio(window.devicePixelRatio);
        this.renderer.setSize(el.clientWidth, el.clientHeight);
        el.appendChild(this.renderer.domElement);

        this.controls = new OrbitControls( this.camera, this.renderer.domElement );
        this.controls.target.z = 1.5; // default, recalculated from points in updatePositions
        
        this.updatePositions();
        this.updateCamera();
        this.updateColors();

        this.frameRequest = window.requestAnimationFrame(() => this.renderFrame());
    },
    destroyed() {
        if(this.controls) this.controls.dispose();
        this.camera = null;
        window.cancelAnimationFrame(this.frameRequest);
    },
    methods: {
        updatePositions() {
            const buffer = [];
            if(this.pointCoordinates && this.pointCoordinates.length) {
                let totalZ = 0;
                for(let [x, y, z] of this.pointCoordinates) {
                    buffer.push(x, y, z);
                    totalZ += z;
                }
                this.controls.target.z = 1.5 * totalZ / this.pointCoordinates.length;
            }
            this.geometry.setAttribute('position', new THREE.Float32BufferAttribute(buffer, 3));
        },
        updateColors() {
            if(!this.currentFrameColors) return;
            this.geometry.setAttribute('color', new THREE.Float32BufferAttribute(this.currentFrameColors, 3));
        },
        updateCamera() {
            this.camera.position.set(...this.cameraPosition);
        },
        renderFrame() {
            this.frameRequest = null;
            this.controls.update();
            this.renderer.render(this.scene, this.camera);

            if(this.camera) {
                this.frameRequest = window.requestAnimationFrame(() => this.renderFrame());
            }
        },
    }
}
</script>

<style lang="scss" scoped>
.RendererViewport {
    display: inline-block;
    min-width: 800px;
    min-height: 560px;
    position: relative;

    &:after {
        pointer-events: none;
        z-index: -1;
        content: '';
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        top: 0;
        outline: 3px solid currentColor;
        opacity: 0.2;
        box-sizing: content-box;
    }
}
</style>