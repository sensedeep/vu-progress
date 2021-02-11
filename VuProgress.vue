<!--
    vu-progress -- Animated progress bar

    Copyright (c) SenseDeep. All Rights Reserved.
-->
<template>
    <div class="progress-bar" :class="classes" :style="styling"></div>
</template>

<script>
import {Vue, Component, Prop} from 'vue-property-decorator'

const Durations = {
    short:  3000,
    medium: 5000,
    long:  10000,
    vlong:  30000,
}
const NapTime = 250

@Component
export default class Progress extends Vue {
    repeat = false
    classes = ''
    styling = {}

    constructor() {
        super()
        app.define({progress: this})
    }

    mounted() {
        this.value = 0
        this.active = false
        this.clear()
    }

    clear() {
        this.value = 0
        this.active = false
        this.styling = {}
        this.classes = ''
    }

    async start(duration) {
        if (typeof duration == 'number') {
            duration *= 1000
        } else if (duration == 'repeat') {
            duration = Durations.short
            this.repeat = true
        } else if (typeof duration != 'number') {
            duration = Durations[duration || 'short']
            if (!duration) {
                duration = Durations.short
            }
        }
        if (duration < NapTime || duration > 9999999999) {
            duration = Durations.short
        }
        if (!this.active) {
            this.active = true
            this.value = 0
            this.inc = duration / NapTime
            this.classes = this.classes.replace(/fade/g, '')
            this.update()
        }
    }

    stop() {
        this.repeat = false
        this.value = 100
        this.classes += ' fade'
        this.active = false
    }

    async update() {
        do {
            while (this.active && this.value <= 100) {
                this.setWidth(this.value)
                await app.delay(NapTime)
                this.value = this.value + (100 / this.inc)
            }
            await app.delay(NapTime)
            this.value = 0
            this.setWidth(0)
        } while (this.repeat);
        this.classes += ' fade'
        this.active = false
    }

    setWidth(width) {
        this.classes = this.classes.replace(' fade', '')
        if (width > 100) {
            width = 100
        }
        if (width > 0) {
            this.styling.transition = 'width 1s linear 0s'
            this.styling.width = `${width}%`
        } else {
            this.styling.transition = 'none'
            this.styling.width = '0%'
        }
        this.$set(this.styling, 'width', this.styling.width)
        this.styling = Object.assign({}, this.styling)
    }
}
</script>

<style lang="scss">
.progress-bar {
    z-index: 201 !important;
    position: absolute;
    background: #2ebcfc;
    top: 48px;
    left: 0;
    height: 6px;
    padding: 0;
    margin: 0;
    border-radius: 0;
    opacity: 1;
    transition: width 1.5s linear;
}
.progress-bar.fade {
    width: 100%;
    animation: progress-bar-fade 0.5s ease-in 0.2s;
    animation-fill-mode: forwards;
}
@keyframes progress-bar-fade {
    0% { opacity: 0.75; }
    100% { opacity: 0; }
}
</style>
