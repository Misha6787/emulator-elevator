<template>
    <div class="elevator__shaft">
        <div
            class="elevator__cabin"
            :style="`
                --distanceFromTop: ${distanceFromTop}px;
                --elevatorCabinSpeed: ${elevatorCabinSpeed}s;
            `"
        >
            <div class="elevator__indicator"></div>
        </div>
    </div>
</template>

<script setup>
    import './ElevatorCabin.scss'

    import {computed} from "vue";

    const props = defineProps({
        activeFloor: {
            default: 1,
            type: Number
        },
        floorCount: {
            default: 1,
            type: Number
        },
        elevatorCabinSpeed: {
            default: 1,
            type: Number
        },
        elevatorCabinHeight: {
            default: 0,
            type: Number
        },
        elevatorCabinStatus: {
            default: true,
            type: Boolean
        }
    })

    const distanceFromTop = computed(() => {
        return (props.floorCount - props.activeFloor) * props.elevatorCabinHeight
    })
</script>