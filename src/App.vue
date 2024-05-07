<template>
    <Elevator
        :elevatorCount="3"
        :floorCount="5"
        :elevatorCabinSize="{
            width: 80,
            height: 100
        }"
        :delayTime="3000"
    />
</template>

<script setup>
    import Elevator from "@/components/Elevator/Elevator.vue";

</script>