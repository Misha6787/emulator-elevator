<template>
    <div
        class="elevator"
        :style="`
            --elevatorCabinWidth: ${elevatorCabinSize.width}px;
            --elevatorCabinHeight: ${elevatorCabinSize.height}px;
            --floorCount: ${props.floorCount};
        `"
    >
        <Floor
            v-for="floor in generatedFloors"
            :key="floor"
            :floor="floor"
            @changeFloor="changeFloor(floor)"
        />
        <ElevatorCabin
            :activeFloor="activeFloor"
            :floorCount="props.floorCount"
            :elevatorCabinSpeed="elevatorCabinSpeed"
            :elevatorCabinHeight="elevatorCabinSize.height"
            :elevatorCabinStatus="elevatorCabinStatus"
        />
    </div>
</template>

<script setup>
    import './Elevator.scss'

    import {computed, ref} from "vue";

    import Floor from "./Floor/Floor.vue";
    import ElevatorCabin from "./ElevatorCabin/ElevatorCabin.vue";

    const props = defineProps({
        floorCount: {
            default: 1,
            type: Number
        }
    })

    // Размер кабины
    const elevatorCabinSize = ref({
        width: 80,
        height: 100
    })

    // Время задержки перед переключением
    const delayTime = 3000

    // Значение прошлого и текущего этажей
    const beforeFloor = ref(1)
    const activeFloor = ref(1)

    // Генерация массива этажей и его переворачивание для правильной последовательности
    const generatedFloors = computed(() => {
        return Array.from({length: props.floorCount}, (_, i) => i + 1).reverse()
    })

    // Вычислиние скорости лифта при переходе между этажами
    const elevatorCabinSpeed = computed(() => {
        return beforeFloor.value - activeFloor.value > 0 ? beforeFloor.value - activeFloor.value : activeFloor.value - beforeFloor.value
    })

    // Статус кабины (готов/не готов к переходу между этажами)
    const elevatorCabinStatus = ref(true)

    // Стек вызовов на этажи
    const elevatorCabinCallStack = []

    // Вызов лифта на этаж
    const changeFloor = (floor, autoChange = false) => {
        // Если мы пытаемся вызвать лифт на этаж, на котором он уже находится
        if (floor === activeFloor.value) return

        // Если кабина не готова к переходу, то добавляем этаж в стек вызовов
        // autoChange нужен для разграничевания автоматической смены этажей и ручной
        if (!elevatorCabinStatus.value && !autoChange) {
            if (!elevatorCabinCallStack.find(item => floor === item)) elevatorCabinCallStack.push(floor)
            return
        }

        // Задаем предыдущий этаж
        beforeFloor.value = activeFloor.value
        // Меняем текущий этаж на следующий
        activeFloor.value = floor

        // Переключаем статус кабины
        elevatorCabinStatus.value = false
        setTimeout(() => {
            if (elevatorCabinCallStack.length > 0) {
                // Запускаем следующий этаж
                changeFloor(elevatorCabinCallStack[0], true)
                // Удаляем этаж из стека
                elevatorCabinCallStack.shift()
            } else {
                elevatorCabinStatus.value = true
            }
            // Задействуем задержку перед следующим вызовом и время перехода между этажами
        }, elevatorCabinSpeed.value * 1000 + delayTime)
    }
</script>