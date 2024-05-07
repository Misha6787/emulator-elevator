<template>
    <div class="elevator__shaft">
        <div
            class="elevator__cabin"
            :style="`
                --distanceFromTop: ${distanceFromTop}px;
                --elevatorCabinSpeed: ${props.elevator.elevatorCabinSpeed}s;
            `"
        >
            <div class="elevator__board">
                <div
                    class="elevator__indicator"
                    :class="!props.elevator.elevatorCabinIndicatorStatus ? 'elevator__indicator_process' : ''"
                >
                </div>
                <div class="elevator__way">
                    <span class="elevator__way-text">{{props.elevator.activeFloor}}</span>
                    <img
                        v-show="!props.elevator.elevatorCabinStatusFloor"
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
        elevator: {
            default: {
                id: 0,
                activeFloor: 1,
                beforeFloor: 1,
                elevatorCabinIndicatorStatus: true,
                elevatorCabinStatus: true,
                elevatorCabinStatusFloor: true,
                elevatorCabinSpeed: 0,
                elevatorCabinCallStack: []
            },
            type: Object
        },
        floorCount: {
            default: 1,
            type: Number
        },
        elevatorCabinHeight: {
            default: 0,
            type: Number
        },
    })

    const distanceFromTop = computed(() => {
        return (props.floorCount - props.elevator.activeFloor) * props.elevatorCabinHeight
    })

    const elevatorRouteDown = computed(() => {
        return props.elevator.beforeFloor > props.elevator.activeFloor
    })
</script>