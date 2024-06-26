<script setup>
import { ref, watch } from "vue";
import Image from "../Shared/Image.vue";
import Line from "./LineComp.vue";
import { sleep, getRandomInt } from "../../utils.js";

const props = defineProps({
    background: {
        type: String,
        default: "gray"
    },
    bgSize: {
        type: String,
        default: "auto"
    },
    imgSrc: {
        type: String,
        default: "/ridge_runners.png"
    },
    lineBg: {
        type: String,
        default: "red"
    },
    started: {
        type: Boolean,
        default: false
    },
    stats: {
        type: Object,
        default: {
            speed: 1,
            accel: 1,
            size: 1
        }
    }
});

const p = {
    HEAD: 1,
    BODY: 2,
    FEET: 3
};

const emit = defineEmits(["imgPositionChanged"]);
const speed = ref(30);
const position = ref(0);
const frame = ref(1);
let interval;

watch(
    () => props.started,
    () => {
        props.started ? move(props.stats) : resetLane();
    }
);

function getRacerClass(stats) {
    let css = 'w-12 sm:w-16 lg:w-28';
    if (stats.speed > 1) css += ' drop-shadow-[0px_0px_10px_rgba(255,255,0,0.9)]';
    if (stats.accel > 1) css += ' drop-shadow-[0px_0px_10px_rgba(255,165,0,0.9)]';
    if (stats.change < 1000) css += ' drop-shadow-[0px_0px_10px_rgba(128,0,128,0.9)]';
    return css;
}

function getRacerImage(frame) {
    return `/duck/${ frame }.png`;
}

/*
* increases position with a certain delay up to 100,
*
* Each position increase:
- changes the frame value connected to img-src, resets it when the value is 10,
- emits the changed position.
*/
async function move(stats) {
    randomAcceleration(stats);
    for (let i = 0; i < 500; i++) {
        if ((i % 4) === 0) frame.value++;
        if (frame.value > 10) frame.value = 1;
        position.value += 0.2;
        await sleep(31 - (speed.value)); // works different in mozilla and chrome
        emit("imgPositionChanged", Math.round(position.value));
    }
    clearInterval(interval);
}

function randomAcceleration(stats) {
    speed.value = getRandomInt(0, 15) + stats.speed; // for randomize starting acceleration
    interval = setInterval(() => {
        stats.speed += stats.accel - 1;
        speed.value = getRandomInt(0, 20) + (stats.speed * stats.size);
    }, (Math.min(getRandomInt(1000, 3000), stats.change)));
}

function resetLane() {
    clearInterval(interval);
    position.value = 0;
    emit("imgPositionChanged", Math.round(position.value));
}
</script>

<template>
    <div class="flex">
        <div
            class="flex w-full pr-12 sm:pr-16 lg:pr-28 bg-center bg-no-repeat"
            :style="{ background: props.background, backgroundSize: props.bgSize }"
        >
            <span class="relative" :style="{ left: position + '%', transform: `scale(${ props.stats.size })` }">
                <Image
                    :class="getRacerClass(props.stats)"
                    :img-src="getRacerImage(frame)"
                ></Image>
            </span>
        <Line class="w-3.5 sm:w-4 lg:w-5" :background="props.lineBg"></Line>
        </div>
        <Line class="ml-auto w-3.5 sm:w-4 lg:w-5" :background="props.lineBg"></Line>
    </div>
</template>
