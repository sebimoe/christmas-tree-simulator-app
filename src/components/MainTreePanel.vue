<template>
    <div class="MainTreePanel">
        <h1>
            <Info bottom info="App created by Sebi Moe, inspired by Matt Parker's Christmas Tree video">
                Christmas Tree Animation Viewer
            </Info>
        </h1>
        <Renderer v-if="simulator" :simulator="simulator" :viewportConfig="viewportConfig" />
        <ControlsBar v-if="simulator" :simulator="simulator" :viewportConfig="viewportConfig" />
    </div>
</template>

<script>
import {
    ChristmasTreeSimulator
} from 'christmas-tree-simulator';

export default {
    data() {
        return {
            simulator: null,
            viewportConfig: {
                maskTexture: "/sphereMask_default.png",
                pointSize: 0.2,
                pointStrength: 0.75,
            }
        }
    },
    mounted() {
        if(!this.simulator) {
            this.simulator = new ChristmasTreeSimulator();
        }
    }
}
</script>

<style lang="scss">
h1 {
    font-size: 1.5rem;
    margin: 0.75rem 0 0;
    text-align: center;
}
</style>