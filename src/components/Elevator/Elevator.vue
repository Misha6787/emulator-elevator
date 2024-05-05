<template>
    <div
        class="elevator"
        :style="`
            --elevatorCabinWidth: ${props.elevatorCabinSize.width}px;
            --elevatorCabinHeight: ${props.elevatorCabinSize.height}px;
            --floorCount: ${props.floorCount};
        `"
    >
        <Floor
            v-for="floor in generatedFloors"
            :key="floor"
            :floor="floor"
            :blockFloors="[...elevatorCabinCallStack, activeFloor]"
            @changeFloor="changeFloor"
        />
        <ElevatorCabin
            :activeFloor="activeFloor"
            :beforeFloor="beforeFloor"
            :floorCount="props.floorCount"
            :elevatorCabinSpeed="elevatorCabinSpeed"
            :elevatorCabinHeight="props.elevatorCabinSize.height"
            :elevatorCabinIndicatorStatus="elevatorCabinIndicatorStatus"
            :elevatorCabinStatusFloor="elevatorCabinStatusFloor"
        />
    </div>
</template>

<script setup>
    import './Elevator.scss'

    import {computed, ref} from "vue";

    import Floor from "./Floor/Floor.vue";
    import ElevatorCabin from "./ElevatorCabin/ElevatorCabin.vue";

    const props = defineProps({
        // Кол-во этажей
        floorCount: {
            default: 5,
            type: Number
        },
        // Размер кабины лифта
        elevatorCabinSize: {
            default: {
                width: 80,
                height: 100
            },
            type: Object
        },
        // Время задержки на этаже
        delayTime: {
            default: 3000,
            type: Number
        }
    })

    // Значение прошлого и текущего этажей
    const beforeFloor = ref(1)
    const activeFloor = ref(1)

    // Статус кабины лифта (готов/не готов к переходу между этажами)
    const elevatorCabinStatus = ref(true)
    // Статус индикатора кабины лифта
    const elevatorCabinIndicatorStatus = ref(true)
    // Статус кабины лифта на момент прибытия на этаж
    const elevatorCabinStatusFloor = ref(true)
    // Стек вызовов на этажи
    const elevatorCabinCallStack = ref([])

    // Генерация массива этажей и его переворачивание для правильной последовательности
    const generatedFloors = computed(() => {
        return Array.from({length: props.floorCount}, (_, i) => i + 1).reverse()
    })

    // Вычислиние скорости лифта при переходе между этажами
    const elevatorCabinSpeed = computed(() => {
        return beforeFloor.value - activeFloor.value > 0 ? beforeFloor.value - activeFloor.value : activeFloor.value - beforeFloor.value
    })

    // Вызов лифта на этаж
    const changeFloor = (floor, autoChange = false) => {
        // Если мы пытаемся вызвать лифт на этаж, на котором он уже находится
        if (floor === activeFloor.value) return

        // Если кабина не готова к переходу, то добавляем этаж в стек вызовов
        // autoChange нужен для разграничевания автоматической смены этажей и ручной
        if (!elevatorCabinStatus.value && !autoChange) {
            if (!elevatorCabinCallStack.value.find(item => floor === item)) {
                elevatorCabinCallStack.value.push(floor)
            }
            return
        }

        // Задаем предыдущий этаж
        beforeFloor.value = activeFloor.value
        // Меняем текущий этаж на следующий
        activeFloor.value = floor

        // Переключаем статус кабины лифта с учетом задержки
        const changeStatusCabinAndCallStack = () => {
            elevatorCabinStatus.value = false
            setTimeout(() => {
                if (elevatorCabinCallStack.value.length > 0) {
                    // Запускаем следующий этаж
                    changeFloor(elevatorCabinCallStack.value[0], true)
                    // Удаляем этаж из стека
                    elevatorCabinCallStack.value.shift()
                } else {
                    elevatorCabinStatus.value = true
                }
                // Задействуем задержку перед следующим вызовом и время перехода между этажами
            }, elevatorCabinSpeed.value * 1000 + props.delayTime)
        }

        // Переключаем статус индикатора кабины лифта на время "отдыха" лифта
        const changeElevatorCabinIndicatorStatus = () => {
            setTimeout(() => {
                elevatorCabinIndicatorStatus.value = false
                setTimeout(() => {
                    elevatorCabinIndicatorStatus.value = true
                }, props.delayTime)
            }, elevatorCabinSpeed.value * 1000)
        }

        // Изменение статуса кабины лифта, когда она уже находится на этаже
        const changeElevatorCabinStatusFloor = () => {
            elevatorCabinStatusFloor.value = false
            setTimeout(() => {
                elevatorCabinStatusFloor.value = true
            }, elevatorCabinSpeed.value * 1000)
        }

        changeStatusCabinAndCallStack()
        changeElevatorCabinIndicatorStatus()
        changeElevatorCabinStatusFloor()
    }
</script>