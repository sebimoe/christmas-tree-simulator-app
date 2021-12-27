<template>
    <div class="ControlsBar">
        <span></span>
        <div class="ControlsBar__section">
            <div class="ControlsBar__input-group">
                <Info info="File containing 3D coordinates of each pixel. By default expects line separated 3-component JSON arrays containing [x,y,z]. (Sometimes called jsonl format).">
                    Pixel coordinates file
                </Info>
                <FilePicker @change="coordinatesFileChanged" :result="coordinateMappingResult" />
            </div>
            <div class="ControlsBar__input-group">
                <Info info="Use alternative format for coordinates file. If checked, expects a CSV file with first 3 columns containing: x, y, z. Maximum 10 total columns allowed in file.">
                    Use CSV format instead.
                    <input type="checkbox" v-model="coordinatesFileUsesCSV" @change="coordinatesFileUsesCSVChanged" />
                </Info>
            </div>
            <div class="ControlsBar__input-group">
                <Info info="CSV file containing brightess data of each pixel for each frame. File should contain columns named R_0, G_0, B_0, R_1, G_1, and so on.">
                    Animation frames file
                </Info>
                <FilePicker @change="animationFileChanged" :result="animationFramesResult" />
            </div>
            <div class="ControlsBar__input-group" >
                <Info info="Use dark theme for the UI. Does not affect tree rendering.">
                    Dark theme
                    <input type="checkbox" v-model="darkTheme" />
                </Info>
            </div>
        </div>
        <div class="ControlsBar__section" v-if="simulator.frameCount">
            <div class="ControlsBar__input-group">
                <Info>
                    Playback controls
                </Info>
                <button @click="simulator.isPlaying ^= true">{{ simulator.isPlaying ? 'Pause' : 'Play' }}</button>
                <input type="range" v-model="simulator.currentProgress" min="0" max="0.99999" step="0.00001" /> 
                <br />
                <div class="equal-columns">
                    <div>
                        Time: {{ simulator.currentTime | fmtTimestamp }}
                    </div>
                    <div v-if="Math.abs(simulator.playbackRate - 1) > 0.005">
                        (Rate: {{~~(simulator.playbackRate*100)}}%)
                    </div>
                </div>
            </div>
            <div class="ControlsBar__input-group">
                <Info info="Adjust the playback speed. Use negative percentages to play backwards.">
                    Playback rate: <input type="range" min="-1" max="2" step="0.01" v-model="simulator.playbackRate" />
                </Info>
            </div>
            <div class="ControlsBar__input-group">
                <Info info="Adjust the size of lights. Try adjusting light strength if the lights saturate at large sizes.">
                    Light size: <input type="range" min="0.05" max="1" step="0.01" v-model="viewportConfig.pointSize" />
                </Info>
            </div>
            <div class="ControlsBar__input-group">
                <Info info="Adjust strength multiplier of lights. Lower this if the lights saturate into white too much.">
                    Light strength: <input type="range" min="0.1" max="1.5" step="0.01" v-model="viewportConfig.pointStrength" />
                </Info>
            </div>
            <div class="ControlsBar__input-group" >
                <Info info="Adjust the mask texture determining shape of each light.">
                    Light shape: 
                    <select v-model="viewportConfig.maskTexture">
                        <option value="/sphereMask_glow.png">Glow</option>
                        <option value="/sphereMask_soft.png">Soft</option>
                        <option value="/sphereMask_default.png" selected>Default</option>
                        <option value="/sphereMask_sharp.png">Sharp</option>
                    </select>
                </Info>
            </div>
        </div>
        <span></span>
    </div>
</template>

<script>
// todo: package up and change path
import {
    CoordinateDecoders,
    CoordinateMapping,
    CsvAnimationDecoder,
} from '../../../christmas-tree-simulator';

export default {
    props: {
        simulator: Object,
        viewportConfig: Object,
    },
    methods: {
        fileChanged(data) {
            console.log("Controls bar: file:", data)
        }
    },
    data() {
        return {
            coordinatesFileUsesCSV: false,
            coordinatesFileContent: null,
            animationFileContent: null,
            coordinateMapping: null,
            coordinateMappingResult: null,
            animationFrames: null,
            animationFramesResult: null,
            darkTheme: false,
        }
    },
    watch: {
        darkTheme() {
            document.body.classList[this.darkTheme ? 'add' : 'remove']('dark-theme');
        }
    },
    created() {
        if (window.matchMedia) {
            const match = window.matchMedia('(prefers-color-scheme: dark)');
            this.darkTheme = match.matches;
            match.addEventListener('change', e => {
                this.darkTheme = e.matches;
            });
        }
    },
    filters: {
        fmtTimestamp(seconds) {
            let minutesStr = '';
            if(seconds >= 60) {
                minutesStr = `${~~(seconds/60)}m `;
                seconds %= 60;
            }
            return `${minutesStr}${~~seconds}s ${(seconds%1).toFixed(3).substr(2)}ms`;
        }
    },
    methods: {
        coordinatesFileChanged(data) {
            this.coordinatesFileContent = data;
            this.recalculateCoordinates(true);
            this.filesUpdated();
        },
        animationFileChanged(data) {
            this.animationFileContent = data;
            this.recalculateAnimation();
            this.filesUpdated();
        },
        coordinatesFileUsesCSVChanged() {
            this.recalculateCoordinates(false);
            this.filesUpdated();
        },
        recalculateCoordinates(attemptCheckboxToggle) {
            if(this.coordinatesFileContent === null) return;
            if(this.tryRecalculateCoordinates(this.coordinatesFileUsesCSV)) {
                if(attemptCheckboxToggle && !this.tryRecalculateCoordinates(!this.coordinatesFileUsesCSV)) {
                    this.coordinatesFileUsesCSV ^= true;
                }
            }
        },
        tryRecalculateCoordinates(coordinatesFileUsesCSV) {
            this.coordinateMapping = new CoordinateMapping(
                coordinatesFileUsesCSV 
                    ? new CoordinateDecoders.CsvCoordinateDecoder()
                    : new CoordinateDecoders.LineDelimitedJsonArrayCoordinateDecoder()
            );
            try {
                this.coordinateMapping.load(this.coordinatesFileContent);
                this.coordinateMappingResult = true;
            }catch(e) {
                console.error("Error while loading coordinate file!", e);
                this.coordinateMappingResult = false;
                return true;
            }
        },
        recalculateAnimation() {  
            if(this.animationFileContent === null) return;

            const frameDecoder = new CsvAnimationDecoder({
                customPixelMapper: ([r, g, b]) => [r/255, g/255, b/255],
                customFrameMapper: pixels => pixels.flat(1), 
            });
            try {
                this.animationFrames = frameDecoder.decode(this.animationFileContent);
                this.animationFramesResult = true;
            }catch(e) {
                console.error("Error while loading animation file!", e);
                this.animationFramesResult = false;
                return;
            }
        },
        filesUpdated() {
            if(this.coordinateMapping == null || this.animationFrames == null) {
                return;
            }

            this.simulator.setData({
                coordinateMapping: this.coordinateMapping, 
                animationFrames: this.animationFrames
            });
        }
    }
}
</script>

<style lang="scss" scoped>
.ControlsBar {
    padding: 1em;
    display: flex;
    justify-content: space-between;

    &__section {
        margin-right: 1em;
    }

    &__input-group {
        margin-bottom: 0.75rem;

        .Info {
            display: block;
        }

        label {
            font-weight: bold;
        }
    }
}

button {
    min-width: 5rem;
    min-height: 1.5rem;
    margin: 0.5rem;
}
.equal-columns {
    display: flex;

    > div {
        flex: 1;
    }
}
</style>
