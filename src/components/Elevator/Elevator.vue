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

    import {computed, onMounted, ref, watch} from "vue";

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

    // Переключаем статус индикатора кабины лифта на время "отдыха" лифта
    const activationIndicator = () => {
        elevatorCabinIndicatorStatus.value = false

        setTimeout(() => {
            elevatorCabinIndicatorStatus.value = true
        }, props.delayTime)
    }

    // Сохранение значений в localStorage
    const setDataInLocalStorage = () => {
        localStorage.setItem('activeFloor', JSON.stringify(activeFloor.value))
        localStorage.setItem('elevatorCabinCallStack', JSON.stringify(elevatorCabinCallStack.value))
    }

    // Получение данных из LocalStorage и запуск перехода этажей на их основе
    const getDataInLocalStorage = () => {
        // Проверяем, есть ли данные в LocalStorage
        if (!localStorage.getItem('activeFloor') || !localStorage.getItem('elevatorCabinCallStack')) return
        activeFloor.value = JSON.parse(localStorage.getItem('activeFloor'))
        elevatorCabinCallStack.value = JSON.parse(localStorage.getItem('elevatorCabinCallStack'))

        // Проверяем, совпадает ли текущий этаж с первым значением в массиве стека вызовов, если да, то удаляем его
        if (activeFloor.value === elevatorCabinCallStack.value[0]) elevatorCabinCallStack.value.shift()

        // Проверяем, есть ли данные в стеке вызовов
        if (elevatorCabinCallStack.value.length > 0) {
            setTimeout(() => {
                changeFloor(elevatorCabinCallStack.value[0], true)
                elevatorCabinCallStack.value.shift()
            }, props.delayTime)

            // Активируем индикатор кабины лифта
            activationIndicator()
        }
    }

    /**
     * Вызов лифта на этаж
     *
     * @param {number} floor — этаж на который должен прибыть лифт
     * @param {boolean} autoChange — параметр, обозначающий, что функция выполняется автоматически
     * @returns {void}
     *
     */
    const changeFloor = (floor, autoChange = false) => {
        // Если мы пытаемся вызвать лифт на этаж, на котором он уже находится
        if (floor === activeFloor.value) return

        // Если кабина не готова к переходу, то добавляем этаж в стек вызовов
        // autoChange нужен для разграничения автоматической смены этажей и ручной
        if (!elevatorCabinStatus.value && !autoChange) {
            if (!elevatorCabinCallStack.value.find(item => floor === item)) elevatorCabinCallStack.value.push(floor)
            return
        }

        // Задаем предыдущий этаж
        beforeFloor.value = activeFloor.value
        // Меняем текущий этаж на следующий
        activeFloor.value = floor

        // Переключаем статус кабины лифта с учетом задержки
        const changeStatusCabinAndCallStack = () => {
            // Переключаем статус кабины лифта
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

        // Переключаем статус индикатора после задержки перехода лифта
        const changeElevatorCabinIndicatorStatus = () => {
            setTimeout(() => {
                activationIndicator()
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

    onMounted(() => {
        getDataInLocalStorage()
    })

    watch([() => activeFloor.value, () => elevatorCabinCallStack.value], () => {
        setDataInLocalStorage()
    }, {deep: true})
</script>