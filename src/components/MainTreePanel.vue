<template>
    <div class="MainTreePanel">
        <Renderer v-if="simulator" :simulator="simulator" :viewportConfig="viewportConfig" />
        <ControlsBar v-if="simulator" :simulator="simulator" :viewportConfig="viewportConfig" />
    </div>
</template>

<script>
// todo: package up and change path
import {
    ChristmasTreeSimulator
} from '../../../christmas-tree-simulator';

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
