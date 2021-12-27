<template>
    <div class="FilePicker" :class="{'FilePicker--dragging': dragging, 'FilePicker--valid': result === true, 'FilePicker--error': result === false}"
        @dragenter.prevent="dragEnter" @dragover.prevent="dragEnter" @dragleave.prevent="dragLeave" @drop.prevent="dropHandler">
        <input type="file" ref="filePicker" @change="changeHandler" />
        <div class="FilePicker__dropLabel">
            Drop to select
        </div>
    </div>
</template>

<script>
export default {
    props: ['result'],
    data() {
        return {
            dragging: false,
        }
    },
    methods: {
        dragEnter() {
            this.dragging = true;
        },
        dragLeave() {
            this.dragging = false;
        },
        dropHandler(e) {
            console.log("dropHandler", e);
            this.dragLeave();
            this.handleFileChanged(e.dataTransfer.files[0]);
        },
        changeHandler() {
            this.handleFileChanged(this.$refs.filePicker.files[0]);
        },
        handleFileChanged(file) {
            if(!file) {
                this.$emit('change', null);
                return;
            }
            const reader = new FileReader();
            reader.addEventListener("load", () => {
                this.$emit('change', reader.result);
            }, false);
            reader.readAsText(file);
        }
    }
}
</script>

<style lang="scss" scoped>
.FilePicker {
    display: inline-block;
    padding: 0.5rem;
    transition: background-color 0.1s;
    position: relative;

    &:after {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(128, 128, 128, 0.6);
        opacity: 0.2;
        z-index: -1;
    }

    input {
        transition: opacity 0.1s;
    }

    &__dropLabel {
        pointer-events: none;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-weight: bold;
        opacity: 0;
        transition: opacity 0.1s;
    }

    &--dragging {
        &:after {
            opacity: 0.5;
        }

        input {
            pointer-events: none;
            opacity: 0;
        }

        .FilePicker__dropLabel {
            opacity: 1;
        }
    }
    
    &--valid:after {
        background: rgba(64, 255, 64);
    }

    &--error:after {
        background: rgba(255, 64, 64);
    }
}
</style>
