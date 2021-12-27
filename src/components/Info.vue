<template>
    <span class="Info" :class="{'Info--bottom': bottom}">
        <span class="Info__content" :title="info"><slot /></span>
        <span class="Info__icon" v-if="info">
            <span class="Info__info">{{info}}</span>
        </span>
    </span>
</template>

<script>
export default {
    props: {
        info: String,
        bottom: Boolean,
    }
}
</script>

<style lang="scss" scoped>
.Info {
    &__info {
        z-index: 100;
        pointer-events: none;
        opacity: 0;
        transition: 0.2s opacity;
        position: absolute;
        width: 250px;
        bottom: 16px;
        left: 50%;
        transform: translateX(-50%);
        padding: 0.5rem;
        border-radius: 0.5rem;
        border: 2px solid rgba(32, 128, 255, 1);
        font-size: 0.85rem;
        background: white;
        color: black;
        line-height: 1.4;
        font-weight: bold;

        .dark-theme & {
            background: #222;
            color: #f0f0f0;
        }
    }

    &--bottom .Info__info {
        bottom: auto;
        top: 16px;
    }

    &__icon {
        position: relative;

        &:hover, &:focus {
            .Info__info {
                display: block;
                opacity: 1;
            }
        }

        &:after {
            content: 'i';
            display: inline-block;
            text-align: center;
            min-width: 0.75rem;
            transform: translateY(-2px);
            padding: 2px;
            border-radius: 32px;
            font-size: 0.65rem;
            background: rgba(32, 128, 255, 0.85);
            color: white;
            font-weight: bold;
        }
    }
}
</style>
