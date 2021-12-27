<template>
    <div class="Renderer">
        <RendererViewport 
            :pointCoordinates="pointCoordinates" 
            :currentFrameColors="currentFrameColors"
            :cameraPosition="cameraPosition"
            :pointSize="Math.sqrt(viewportConfig.pointSize * viewportConfig.pointSize)"
            :pointStrength="viewportConfig.pointStrength"
            :maskTexture="viewportConfig.maskTexture"
        />
    </div>
</template>

<script>
export default {
    props: {
        simulator: Object,
        viewportConfig: Object,
    },
    data() {
        return {
            mainLoopTimeout: null,
            lastTickDate: null,
            pointCoordinates: null,
            currentFrameColors: null,
            cameraPosition: [-2, -2, 3],
        };
    },
    mounted() {
        this.lastTickDate = new Date();
        this.mainLoopTimeout = window.requestAnimationFrame(() => this.animationFrameCallback());
    },
    destroyed() {
        window.cancelAnimationFrame(this.mainLoopTimeout);
    },
    methods: {
        animationFrameCallback() {
            try {
                const deltaTimeMs = new Date() - this.lastTickDate;
                this.mainLoopTick(deltaTimeMs / 1000);
            }catch(e) {
                console.log("Render loop error!", e);
            }finally{
                this.lastTickDate = new Date();
                this.mainLoopTimeout = window.requestAnimationFrame(() => this.animationFrameCallback());
            }
        },
        mainLoopTick(deltaTime) {
            this.simulator.processTimeAdvance(deltaTime);
            if(this.simulator.clearDirty()) {
                this.renderFrame();
            }
        },
        renderFrame() {
            if(!this.simulator.coordinateMapping) {
                return;
            }
            
            if(this.simulator.clearGeometryDirty()) {
                this.pointCoordinates = this.simulator.coordinateMapping.allCoords();
            }

            this.currentFrameColors = this.simulator.animationFrames[this.simulator.currentFrameIdx];
        }
    }
}
</script>

<style lang="scss" scoped>
.Renderer {
    text-align: center;
    padding: 1rem;
}
</style>
