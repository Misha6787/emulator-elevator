<template>
    <div
        class="elevator"
        :style="`
            --elevatorCabinWidth: ${props.elevatorCabinSize.width}px;
            --elevatorCabinHeight: ${props.elevatorCabinSize.height}px;
            --floorCount: ${props.floorCount};
            --elevatorCount: ${props.elevatorCount};
        `"
    >
        <Floor
            v-for="floor in generatedFloors"
            :key="floor"
            :floor="floor"
            :blockFloors="blockFloors"
            @changeFloor="changeFloor"
        />
        <div class="elevators__container">
            <ElevatorCabin
                v-for="elevator in elevators"
                :key="elevator.id"
                :elevator="elevator"
                :floorCount="props.floorCount"
                :elevatorCabinHeight="props.elevatorCabinSize.height"
            />
        </div>
    </div>
</template>

<script setup>
    import './Elevator.scss'

    import {computed, onMounted, ref} from "vue";

    import Floor from "./Floor/Floor.vue";
    import ElevatorCabin from "./ElevatorCabin/ElevatorCabin.vue";

    const props = defineProps({
        // Кол-во кабин лифта
        elevatorCount: {
            default: 1,
            type: Number
        },
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

    /**
     * Массив кабин лифта
     *
     * @type {Ref<UnwrapRef<*[]>>}
     *
     */
    const elevators = ref([])

    // Генерируем массив кабин лифта
    for (let i = 0; i < props.elevatorCount; i++) {
        elevators.value.push({
            id: i,
            activeFloor: 1,
            beforeFloor: 1,
            elevatorCabinIndicatorStatus: true,
            elevatorCabinStatus: true,
            elevatorCabinStatusFloor: true,
            elevatorCabinSpeed: 0,
            elevatorCabinCallStack: []
        })
    }

    // Генерация массива этажей и его переворачивание для правильной последовательности
    const generatedFloors = computed(() => {
        return Array.from({length: props.floorCount}, (_, i) => i + 1).reverse()
    })

    // Массив заблокированных кнопок
    const blockFloors = computed(() => {
        return [...elevators.value.flatMap(obj => obj.elevatorCabinCallStack), ...elevators.value.map(obj => obj.activeFloor)]
    })

    // Переключаем статус индикатора кабины лифта на время "отдыха" лифта
    const activationIndicator = (elevator) => {
        elevator.elevatorCabinIndicatorStatus = false

        setTimeout(() => {
            elevator.elevatorCabinIndicatorStatus = true
        }, props.delayTime)
    }

    // Сохранение значение кабин лифт в localStorage
    const setDataInLocalStorage = () => {
        localStorage.setItem('elevators', JSON.stringify(elevators.value))
    }

    // Получение данных из LocalStorage и запуск перехода этажей на их основе
    const getDataInLocalStorage = () => {
        // Проверяем, есть ли данные в LocalStorage
        if (!localStorage.getItem('elevators')) return

        elevators.value = JSON.parse(localStorage.getItem('elevators'))

        elevators.value.forEach(elevator => {
            // Переводим текущюю кабину лифта в статус готовности к переходу
            elevator.elevatorCabinStatus = true
            elevator.elevatorCabinStatusFloor = true
            elevator.elevatorCabinIndicatorStatus = true

            // Проверяем, есть ли данные в стеке вызовов
            if (elevator.elevatorCabinCallStack.length > 0) {
                setTimeout(() => {
                    changeFloor(elevator.elevatorCabinCallStack[0], true, elevator)
                    elevator.elevatorCabinCallStack.shift()
                }, props.delayTime)

                // Активируем индикатор кабины лифта
                activationIndicator(elevator)
            }
        })
    }

    // Функция вычисления времени перехода между 2 этажами
    const findTransitionTime = (first, second) => {
        return first > second ? first - second : second - first
    }

    // Получаем кабину лифта, наиболее подходящую для выбранного этажа
    const getCurrentElevator = (floor) => {
        // Фильтруем кабины лифта, у которых есть выбранный этаж в стеке вызовов, или он уже находится на этаже
        const currentElevators = elevators.value.filter(item => item.activeFloor !== floor && !item.elevatorCabinCallStack.find(fl => fl === floor))

        // Проверка на то, есть ли вообще лифты на которые могут приехать на этаж
        if (currentElevators.length > 0) {
            const speedFinishCallStacks = []

            // Поиск кабин с наименьшим временем перехода между этажами с учетом времени перехода на выбранный этаж
            currentElevators.forEach(elevator => {
                let countSecond = 0
                elevator.elevatorCabinCallStack.forEach((item, index, arr) => {
                    if (index === arr.length - 1) return
                    const secondItem = arr[index + 1]
                    countSecond += findTransitionTime(item, secondItem)
                });
                speedFinishCallStacks.push({
                    ...elevator,
                    transitionTime: countSecond + (findTransitionTime(elevator.activeFloor, floor) + (elevator.elevatorCabinCallStack.length * (props.delayTime / 1000) - 1))
                })
            })

            // Находим лифта с наименьшим временем
            const currentElevator = speedFinishCallStacks.reduce((arrFirst, arrSecond) => {
                return arrFirst.transitionTime > arrSecond.transitionTime ? arrSecond : arrFirst
            }, speedFinishCallStacks[0])

            // Получаем и отдаем кабину лифта
            return elevators.value.find(elevator => elevator.id === currentElevator.id);
        } else {
            return null
        }
    }

    /**
     * Вызов лифта на этаж
     *
     * @param {number} floor — этаж на который должен прибыть лифт
     * @param {boolean} autoChange — параметр, обозначающий, что функция выполняется автоматически
     * @param {array} saveElevator
     * @returns {void}
     *
     */
    const changeFloor = (floor, autoChange = false, saveElevator = null) => {
        // Если передана кабина лифта, ее и используем
        const currentElevator = saveElevator ?? getCurrentElevator(floor);

        // Если кабина лифта не наидена, ничего не делаем
        if (!currentElevator) return

        // Если мы пытаемся вызвать лифт на этаж, на котором он уже находится
        if (floor === currentElevator.activeFloor) return

        // Если кабина не готова к переходу, то добавляем этаж в стек вызовов
        // autoChange нужен для разграничения автоматической смены этажей и ручной
        if (!currentElevator.elevatorCabinStatus && !autoChange) {
            if (!currentElevator.elevatorCabinCallStack.find(item => floor === item)) currentElevator.elevatorCabinCallStack.push(floor)
            return
        }

        // Задаем предыдущий этаж
        currentElevator.beforeFloor = currentElevator.activeFloor
        // Меняем текущий этаж на следующий
        currentElevator.activeFloor = floor

        // Вычислиние скорости лифта при переходе между этажами
        currentElevator.elevatorCabinSpeed = findTransitionTime(currentElevator.beforeFloor, currentElevator.activeFloor)

        // Переключаем статус кабины лифта с учетом задержки
        const changeStatusCabinAndCallStack = () => {
            // Переключаем статус кабины лифта
            currentElevator.elevatorCabinStatus = false

            setTimeout(() => {
                if (currentElevator.elevatorCabinCallStack.length > 0) {
                    // Запускаем следующий этаж
                    changeFloor(currentElevator.elevatorCabinCallStack[0], true, currentElevator)
                    // Удаляем этаж из стека
                    currentElevator.elevatorCabinCallStack.shift()
                } else {
                    currentElevator.elevatorCabinStatus = true
                }
                // Задействуем задержку перед следующим вызовом и время перехода между этажами
            }, currentElevator.elevatorCabinSpeed * 1000 + props.delayTime)
        }

        // Переключаем статус индикатора после задержки перехода лифта
        const changeElevatorCabinIndicatorStatus = () => {
            setTimeout(() => {
                setDataInLocalStorage()
                activationIndicator(currentElevator)
            }, currentElevator.elevatorCabinSpeed * 1000)
        }

        // Изменение статуса кабины лифта, когда она уже находится на этаже
        const changeElevatorCabinStatusFloor = () => {
            currentElevator.elevatorCabinStatusFloor = false
            setTimeout(() => {
                currentElevator.elevatorCabinStatusFloor = true
            }, currentElevator.elevatorCabinSpeed * 1000)
        }

        changeStatusCabinAndCallStack()
        changeElevatorCabinIndicatorStatus()
        changeElevatorCabinStatusFloor()

        setDataInLocalStorage()
    }

    onMounted(() => {
        getDataInLocalStorage()
    })
</script>
