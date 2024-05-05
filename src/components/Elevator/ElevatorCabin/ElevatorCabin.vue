<template>
    <div class="elevator__shaft">
        <div
            class="elevator__cabin"
            :style="`
                --distanceFromTop: ${distanceFromTop}px;
                --elevatorCabinSpeed: ${props.elevatorCabinSpeed}s;
            `"
        >
            <div class="elevator__board">
                <div
                    class="elevator__indicator"
                    :class="!props.elevatorCabinIndicatorStatus ? 'elevator__indicator_process' : ''"
                >
                </div>
                <div class="elevator__way">
                    <span class="elevator__way-text">{{props.activeFloor}}</span>
                    <img
                        v-show="!props.elevatorCabinStatusFloor"
                        src="../../../assets/arrow.png"
                        class="elevator__way-arrow"
                        :class="elevatorRouteDown ? 'elevator__way-arrow_down' : ''"
                        alt="elevator way arrow"
                    >
                </div>
            </div>
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
        beforeFloor: {
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
        elevatorCabinIndicatorStatus: {
            default: true,
            type: Boolean
        },
        elevatorCabinStatusFloor: {
            default: true,
            type: Boolean
        }
    })

    const distanceFromTop = computed(() => {
        return (props.floorCount - props.activeFloor) * props.elevatorCabinHeight
    })

    const elevatorRouteDown = computed(() => {
        return props.beforeFloor > props.activeFloor
    })
</script>