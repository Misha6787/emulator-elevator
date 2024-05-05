<template>
    <div class="floor">
        <div class="floor__content">
            <span>{{ props.floor }}</span>
            <button
                class="floor__button"
                :class="props.blockFloors.find(item => item === props.floor) ? 'floor__button_error' : ''"
                @click="emit('changeFloor', props.floor)"
            >
                <img class="floor__button-img" src="../../../assets/btn.png" alt="btn">
            </button>
        </div>
    </div>
</template>

<script setup>
    import './Floor.scss'

    const props = defineProps({
        floor: {
            default: 1,
            type: Number
        },
        blockFloors: {
            default: [],
            type: Array
        }
    })

    const emit = defineEmits(['changeFloor'])

</script>