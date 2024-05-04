<template>
    <div class="floor">
        <div class="floor__content">
            {{ props.floor }}
            <button class="floor__button" @click="emit('changeFloor')">
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
        }
    })

    const emit = defineEmits(['changeFloor'])
</script>