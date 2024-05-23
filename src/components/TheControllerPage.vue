<template>
    <div class="page-content__container">
        <div>
            <div class="controller-page__info-container">
                <div class="controller-page__info-block">
                    <p>название</p>
                    <span>{{ controllerInfo.name }}</span>
                </div>
                <div class="controller-page__info-block">
                    <p>cерийный номер</p>
                    <span>{{ controllerInfo.sn }}</span>
                </div>
                <div class="controller-page__info-block">
                    <p>тип контроллера</p>
                    <span>{{ controllerInfo.device_type.device_type }}</span>
                </div>
                <div class="controller-page__info-block">
                    <p>уровень сигнала</p>
                    <span>{{ lastReleaseSignal }}</span>
                </div>
                <div class="controller-page__info-block">
                    <p>дата последнего выхода</p>
                    <span>{{ lastReleaseDate }}</span>
                </div>
            </div>
            <div class="account-separator"></div>
        </div>
        <nav className="controller-nav">
            <div class="controller-nav__btns">
                <button @click="dashBoardOn()"
                    v-bind:class="{ controller_btn_active: btns.dashBoardActive }">Дашборд</button>
                <button @click="settingsOn()"
                    v-bind:class="{ controller_btn_active: btns.settingsActive }">Настройки</button>
                <button @click="dataOn()" v-bind:class="{ controller_btn_active: btns.dataActive }"> Детальные
                    данные</button>
            </div>
            <div class="data-filter__container">
                <div className="datetext">{{ dateText }}</div>
                <div class="controller-nav__data-filter">
                    <button class="dropdown__button dropdown__button-data-filter" @click="chooseAccessLevel()"
                        v-bind:class="{ opened_button: selectortimeVisible }">{{ selectortimeContent }}<i
                            class="white_delta"
                            v-bind:class="{ inverted_white_delta: selectortimeVisible }"></i></button>
                    <ul class="dropdown__list dropdown__list-data-filter"
                        v-bind:class="{ dropdown__list_visible: selectortimeVisible }">
                        <div class="datafilter-separator"></div>
                        <li class="dropdown__list-item" @click="showDay()">Последний день</li>
                        <li class="dropdown__list-item" @click="showWeek()">Последние 7 дней</li>
                        <li class="dropdown__list-item" @click="showMonth()">Последние 30 дней</li>
                        <li class="dropdown__list-item" @click="showAllTime()">Всё время</li>
                    </ul>
                </div>
            </div>
        </nav>
        <div v-if="btns.loading" class="loading">
            <div>
                <div class="loader"></div>
            </div>
        </div>
        <div class="dashboard-container" v-if="btns.dashBoardActive">
            <div class="info-block__first">
                <div class="info-block__half">
                    <div class="pies-block">
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Напряжение АКБ</h4>
                                <ThePieVoltage v-if="batCChart" :controllerInfoStorage="receivedData"
                                    :lastResult="lastResult" />
                            </div>
                        </div>
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Уровень заряда АКБ</h4>
                                <ThePieChart v-if="batCChart" :controllerInfoStorage="receivedData"
                                    :lastResult="lastResult" />
                            </div>
                        </div>
                    </div>
                    <div class="pies-block">
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Сгенерированно энергии</h4>
                                <p class="pie-last-time">{{ dateText }}</p>
                                <ThePieChartTwo v-if="visibleChart" :controllerInfoStorage="receivedData" />
                            </div>
                        </div>
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Потрачено энергии</h4>
                                <p class="pie-last-time">{{ dateText }}</p>
                                <ThePieChartThree v-if="visibleChart" :controllerInfoStorage="receivedData" />
                            </div>
                        </div>
                    </div>
                </div>
                <div class="info-block__block info-block__half info-block__errors">
                    <div class="info-block__errors-container">
                        <h4>ошибки</h4>
                    </div>
                    <div className="info-line info-line__title info-line__title-errors">
                        <div class="measured_at measured-at__dashboard-errors">дата/время</div>
                        <div class="measured-at__dashboard-code">Описание</div>
                    </div>
                    <div class="controller-data__dashboard-errors">
                        <div className="info-line info-line__table" v-for="info in lastErrors" :key="info">
                            <div class="measured_at measured-at__dashboard-errors">{{ info.measured_at }}</div>
                            <div class="measured-at__dashboard-code">{{ info.name }}</div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="info-block__first">
                <div class="info-block__half">
                    <div class="info-block__block info-block__block-current ">
                        <h4>ток</h4>
                        <TheBarChart v-if="visibleChart" :controllerInfoStorage="receivedData" />
                    </div>
                </div>
                <div class="info-block__half">
                    <div class="info-block__block">
                        <h4>Напряжение</h4>
                        <TheVoltageChart v-if="visibleChart" :controllerInfoStorage="receivedData" />
                    </div>
                </div>
            </div>
            <div class="info-block__first">
                <div class="info-block__half dashboard-map">
                    <div class="info-block__block">
                        <div id="map-dashboard" style="width: 100%; height: 100%;"></div>
                    </div>
                </div>
                <div class="info-block__half dashboard-table">
                    <div v-if="thereIsData" class="there-is-data">Нет данных за период</div>
                    <div class="info-block__block dashboard-spectable">
                        <div class="info-block__half-title">
                            <h4 class="dashboard-table__title">Последние данные</h4><button class="save-btn more-btn"
                                @click="dataOn()">больше
                                →</button>
                        </div>
                        <div className="info-line info-line__title info-line__title-dashboard">
                            <div class="measured_at measured-at__dashboard">дата/время</div>
                            <div>Напряжение PV(В)</div>
                            <div>Напряжение АКБ(В)</div>
                            <div>Напряжение нагрузки(В)</div>
                            <div>Ток PV(А)</div>
                            <div>Ток АКБ(А)</div>
                            <div>Ток нагрузки(А)</div>
                        </div>
                        <div class="controller-data__dashboard">
                            <div className="info-line info-line__table" v-for="info in receivedData" :key="info">
                                <div class="measured_at measured-at__dashboard">{{ info.created_at }}</div>
                                <div>{{ info.pv_v }}</div>
                                <div>{{ info.bat_v }}</div>
                                <div>{{ info.load_v }}</div>
                                <div>{{ info.pv_i }}</div>
                                <div>{{ info.bat_i }}</div>
                                <div>{{ info.load_i }}</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="controller-settings" v-if="btns.settingsActive">
            <div class="controller-info">
                <p><span>ID</span> {{ controllerInfo.id }}</p>
                <p><span>Название</span> {{ controllerInfo.name }}</p>
                <p><span>Название</span> {{ controllerInfo.name }}</p>
                <p><span>Описание</span> {{ controllerInfo.description }}</p>
                <p><span>Серийный номер</span> {{ controllerInfo.sn }}</p>
                <p><span>Пин-код</span> {{ controllerInfo.pin }}</p>
                <p><span>Тип устройства</span> {{ controllerInfo.device_type.device_type }}</p>
                <p><span>Дата создания</span> {{ controllerInfo.created_at }}</p>
            </div>
            <div class="controller-map">Карта</div>
        </div>
        <div v-if="btns.dataActive" class="dashboard-table">
            <div v-if="thereIsData" class="there-is-data">Нет данных за период</div>
            <div className="info-line info-line__title">
                <div class="measured_at">дата/время</div>
                <div>Напряжение PV(В)</div>
                <div>Напряжение АКБ(В)</div>
                <div>Напряжение нагрузки(В)</div>
                <div>Ток PV(А)</div>
                <div>Ток АКБ(А)</div>
                <div>Ток нагрузки(А)</div>
            </div>
            <div class="controller-data">
                <div className="info-line" v-for="info in receivedData" :key="info">
                    <div class="measured_at">{{ info.created_at }}</div>
                    <div>{{ info.pv_v }}</div>
                    <div>{{ info.bat_v }}</div>
                    <div>{{ info.load_v }}</div>
                    <div>{{ info.pv_i }}</div>
                    <div>{{ info.bat_i }}</div>
                    <div>{{ info.load_i }}</div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios';
import TheBarChart from './charts/TheBarChart.vue';
import TheVoltageChart from './charts/TheVoltageChart.vue';
import ThePieChart from './charts/ThePieChart.vue';
import ThePieChartTwo from './charts/ThePieChartTwo.vue';
import ThePieChartThree from './charts/ThePieChartThree.vue';
import ThePieVoltage from './charts/ThePieVoltage.vue';

export default {
    components: {
        TheBarChart,
        TheVoltageChart,
        ThePieChart,
        ThePieChartTwo,
        ThePieChartThree,
        ThePieVoltage
    },
    props: {
        controllerId: Text
    },
    data() {
        return {
            btns: {
                dashBoardActive: true,
                settingsActive: false,
                dataActive: false,
                loading: true
            },
            batCChart: false,
            selectortimeContent: '',
            selectortimeVisible: false,

            // даты
            dateEnd: '',
            dateStart: '',
            dateText: '',

            controllerInfo: {
                account: '',
                created_at: "",
                description: "Описание",
                device_type: { id: '', device_type: '' },
                id: '',
                installer: null,
                name: "Устройство",
                pin: '',
                sn: "",
                status: {},
                updated_at: ""
            },

            controllerInfoStorage: [], // данные за период
            receivedData: [],
            smallControllerInfoStorage: [],
            indicatorSearchLastEntry: false,
            lastResult: [],
            lastReleaseDate: ' ', // последний выход на связь,
            lastReleaseSignal: ' ',

            visibleChart: false,
            thereIsData: false,

            lastErrors: [],

            coord: null, // координаты для дашборда
            coordinates: { latitude: 55.76, longitude: 37.64 }
        }
    },
    methods: {
        dashBoardOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.dashBoardActive = true;
        },
        settingsOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.settingsActive = true;
        },
        dataOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.dataActive = true;
        },
        chooseAccessLevel() {
            this.selectortimeVisible = !this.selectortimeVisible;
        },
        twoDigits(num) {
            return ('0' + num).slice(-2);
        },
        showDay() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime());
            // let datestart = new Date(today.getTime() - (24 * 60 * 60 * 1000));
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;

            this.selectortimeContent = 'Последний день';
            this.getControllerData();
        },
        showWeek() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime() - (7 * 24 * 60 * 60 * 1000));
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;

            this.selectortimeContent = 'Последние 7 дней';
            this.getControllerData();
        },
        showMonth() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime() - (30 * 24 * 60 * 60 * 1000));
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;

            this.selectortimeContent = 'Последние 30 дней';
            this.getControllerData();
        },
        showAllTime() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            this.dateStart = '2024-02-01';

            this.selectortimeContent = 'Всё время';
            this.getControllerData();
        },
        getControllerData() {
            this.indicatorSearchLastEntry = false;
            this.btns.loading = true; // показать загрузку
            this.visibleChart = false; // скрыть графики
            this.thereIsData = false; // скрыть блок «Нет данных за этот период»

            this.selectortimeVisible = false; // закрыть селектор

            let start = this.dateStart.split('-');
            let end = this.dateEnd.split('-'); // ['2024', '04', '17']

            this.dateText = `${start[2]}.${start[1]} – ${end[2]}.${end[1]}`;

            // ?date_start=2024-02-01
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=${this.dateStart}&limit=100000`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.controllerInfoStorage = response.data;

                        if (this.controllerInfoStorage.length === 0) {
                            this.indicatorSearchLastEntry = true; // найти последнюю запись
                            this.thereIsData = true; // показать блок «Нет данных за этот период»
                        }

                        if (this.controllerInfoStorage.length !== 0) {
                            this.receivedData = this.controllerInfoStorage; //исходный ответ с сервера (для графиков и последнего выхода на связь)
                            let date = this.receivedData[0].created_at;
                            let formatDate = date.split(',');

                            this.receivedData.forEach((el) => {
                                let a = el.created_at.split(',')
                                el.created_at = a[0].slice(0, -5) + ' ' + a[1].slice(0, -3); // дата без вывода секунд
                            })

                            this.lastReleaseDate = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                            this.lastReleaseSignal = this.receivedData[0].dbi;
                        }
                        this.correctControllerData();
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        correctControllerData() {
            this.batCChart = false;
            this.visibleChart = true;

            if (this.indicatorSearchLastEntry) {
                this.searchLastEntry();
            } else {
                this.batCChart = true;
            }
            this.getErrors();
        },
        getErrors() {
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/event/?type=3&limit=20`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastErrors = response.data;

                        if (this.lastErrors.length > 10) {
                            this.lastErrors = this.lastErrors.slice(-17);
                        }

                        this.lastErrors.forEach((el) => {
                            let date = el.measured_at;
                            let formatDate = date.split(',');
                            el.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                        })
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
            if (!document.getElementById('map-dashboard').firstChild) {
                this.getCoords();
            } else {
                this.btns.loading = false;
            }
        },
        getCoords() {
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/gps/`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.coord = this.convertCoordinates(response.data[0].value);
                        this.drawControllerMap();
                    }
                }).catch((error) => {
                    console.log(error); // обработка ошибки
                    this.drawControllerMap();
                })
        },
        drawControllerMap() {
            if (this.coord) {
                if ((this.coord.latitude !== 0)) {
                    this.coordinates = this.coord;
                }
            }
            ymaps.ready(() => {
                const dashMap = new ymaps.Map('map-dashboard', {
                    center: [this.coordinates.latitude, this.coordinates.longitude],
                    zoom: 10
                });

                if (this.coord) {
                    let myPlacemark = new ymaps.Placemark([this.coord.latitude, this.coord.longitude], {
                        balloonContent: `${this.controllerId}`
                    });
                    dashMap.geoObjects.add(myPlacemark);
                }
            });

            this.btns.loading = false;
        },
        convertCoordinates(coordinates) {
            const parts = coordinates.split(',');

            const latDegreesMinutes = parts[0];
            const lonDegreesMinutes = parts[2];

            const latDegrees = parseInt(latDegreesMinutes.substr(0, 2));
            const latMinutes = parseFloat(latDegreesMinutes.substr(2));

            const lonDegrees = parseInt(lonDegreesMinutes.substr(0, 3));
            const lonMinutes = parseFloat(lonDegreesMinutes.substr(3));

            const latDirection = parts[1];
            const lonDirection = parts[3];

            let latDecimal = latDegrees + latMinutes / 60;
            let lonDecimal = lonDegrees + lonMinutes / 60;

            if (latDirection === 'S') {
                latDecimal = -latDecimal;
            }

            if (lonDirection === 'W') {
                lonDecimal = -lonDecimal;
            }

            return { latitude: latDecimal, longitude: lonDecimal };
        },
        searchLastEntry() {
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=2024-02-01&limit=1`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastResult = response.data;

                        let date = this.lastResult[0].created_at;
                        let formatDate = date.split(',');
                        this.lastResult[0].created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                        this.lastReleaseDate = this.lastResult[0].created_at;

                        this.batCChart = true;
                        this.lastReleaseSignal = '-';
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        }
    },
    mounted() {
        this.showDay();

        // Информация об устройстве
        axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                if (response.status === 200) {
                    this.controllerInfo = response.data;
                }
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    }
}
</script>

<style>
@keyframes mulShdSpin {

    0%,
    100% {
        box-shadow: 0 -3em 0 0.2em,
            2em -2em 0 0em, 3em 0 0 -1em,
            2em 2em 0 -1em, 0 3em 0 -1em,
            -2em 2em 0 -1em, -3em 0 0 -1em,
            -2em -2em 0 0;
    }

    12.5% {
        box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
            3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
            -2em 2em 0 -1em, -3em 0 0 -1em,
            -2em -2em 0 -1em;
    }

    25% {
        box-shadow: 0 -3em 0 -0.5em,
            2em -2em 0 0, 3em 0 0 0.2em,
            2em 2em 0 0, 0 3em 0 -1em,
            -2em 2em 0 -1em, -3em 0 0 -1em,
            -2em -2em 0 -1em;
    }

    37.5% {
        box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
            3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
            -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
    }

    50% {
        box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
            3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
            -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
    }

    62.5% {
        box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
            3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
            -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
    }

    75% {
        box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
            3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
            -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
    }

    87.5% {
        box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
            3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
            -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
    }
}

.loading {
    display: flex;
    justify-content: center;
    background-color: #EEEEEE;
    height: 110vh;
    width: 100%;
    position: absolute;
    left: 0;
    z-index: 98;
}

.loader {
    margin-top: 200px;
    width: 70px;
    aspect-ratio: 1;
    display: grid;
}

.loader::before,
.loader::after {
    content: "";
    grid-area: 1/1;
    --c: no-repeat radial-gradient(farthest-side, #2482c5 95%, #b3373700);
    background:
        var(--c) 50% 0,
        var(--c) 50% 100%,
        var(--c) 100% 50%,
        var(--c) 0 50%;
    background-size: 12px 12px;
    animation: l12 1s infinite;
}

.loader::before {
    margin: 4px;
    filter: hue-rotate(45deg);
    background-size: 8px 8px;
    animation-timing-function: linear
}

@keyframes l12 {
    100% {
        transform: rotate(.5turn)
    }
}

.info-line {
    display: flex;
    width: 100%;
    justify-content: space-between;
    font-size: 14px;
}

.info-line:hover {
    background-color: #e0e4ed;
}

.info-line__title {
    margin-bottom: 8px;
}

.info-line__title:hover {
    background-color: transparent;
}

.info-line div {
    text-align: center;
    width: 115px;
    height: 32px;
    display: flex;
    justify-content: center;
    align-items: center;
}

.measured_at {
    width: 150px !important;
    font-weight: 500 !important;
}

.controller-nav {
    margin-bottom: 24px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.controller-nav button {
    font-weight: 500;
    font-size: 16px;
    line-height: 125%;
    color: #293b5f;
    margin-right: 32px;
}

.controller_btn_active {
    font-weight: 700 !important;
}

.controller-nav__data-filter {
    width: 208px;
    position: relative;
    z-index: 99;
}

.dropdown__button-data-filter {
    margin: 0;
    height: 40px;
    color: #F8F6F4 !important;
}

.dropdown__list-data-filter {
    top: 40px;
}

.datafilter-separator {
    margin: 0px 16px 8px 16px;
}

.dropdown__list-data-filter li {
    margin: 0 6px 4px;
}

.dropdown__button-data-filter i {
    width: 16px;
    height: 16px;
}

.data-filter__container {
    display: flex;
}

.datetext {
    display: flex;
    align-items: center;
    margin-right: 14px;
    color: #294b8e;
}

.controller-settings {
    display: flex;
    justify-content: space-between;
}

.controller-info span {
    font-weight: 500;
}

.controller-info {
    margin-right: 24px;
}

.controller-info,
.controller-map {
    width: 50%;
    background-color: #F8F6F4;
    border-radius: 8px;
    padding: 32px;
    font-weight: 400;
    font-size: 14px;
    line-height: 129%;
    color: #0E1626;
}

.controller-nav__btns button {
    background-color: transparent;
}

.controller-data {
    height: 50vh;
    overflow-y: scroll;
}

.info-block__first {
    display: flex;
    justify-content: space-between;
    margin-bottom: 24px;
}

.info-block__block {
    background-color: #F8F6F4;
    border-radius: 8px;
    padding: 16px;
    font-weight: 400;
    font-size: 14px;
    color: #0E1626;
}

.info-block__half {
    width: 49%;
}

.info-block__block-current {
    padding: 16px;
}

.info-block__errors {
    padding: 0px;
}

.info-block__errors-container {
    padding: 16px;
}

.dashboard-container h4 {
    font-weight: 500;
    margin: 0;
    font-size: 14px;
    text-transform: uppercase;
    text-align: center;
}

.info-line__title-dashboard {
    font-size: 10px;
}

.measured-at__dashboard {
    width: 100px !important;
    font-size: 10px !important;
}

.measured-at__dashboard-errors {
    width: 150px !important;
}

.measured-at__dashboard-code {
    width: 200px !important;
}

.info-line__title-errors {
    font-size: 13px;
    justify-content: flex-start;
}

.info-line__title-errors div {
    width: 134px !important;
    font-weight: 500;
}

.info-line__table div {
    font-size: 12px;
}

.controller-data__dashboard {
    overflow: hidden;
    height: 230px;
}

.controller-data__dashboard-errors {
    overflow-y: hidden;
    overflow-x: hidden;
    padding: 0 0 0 16px;
}

.info-block__second div {
    width: 28%;
    padding: 24px;
}

.info-block__second div:last-child {
    margin-right: 0px;
}

.info-block__half-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
}

.info-block__half-title h4 {
    margin: 0;
}

.more-btn {
    border-radius: 8px;
    padding: 6px 12px;
    margin-right: 0px;
}

.pie-container canvas {
    padding: 24px;
    width: 100% !important;
    height: 100% !important;
}

.info-block_line-charts {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.info-block_line-charts div {
    height: 50%;
    display: flex;
    justify-content: center;
    padding: 16px;
}

.info-line__title-dashboard:hover {
    background-color: transparent;
}

.charge-level {
    margin-bottom: 16px !important;
}

.info-block_line-charts div {
    display: flex;
    flex-direction: column;
}

.pie-container {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.pie-value {
    position: absolute;
    padding: 0 !important;
    font-size: 40px;
    width: 100%;
    top: 44%;
    text-align: center;
}

.pie-energy {
    top: 34%;
}

.pie-voltage {
    top: 40%;
}

.pie-value p {
    margin: 0;
    font-size: 20px;
}

.pie-last-time {
    margin: 0;
    text-align: center;
    color: #293b5f;
}

.controller-page__info-block {
    border-radius: 8px;
    width: 30%;
    box-shadow: 3px 3px 6px 0 rgba(41, 75, 142, 0.5);
    background: linear-gradient(90deg, #294b8e 27%, #2384c5 100%);
    font-weight: 400;
    font-size: 14px;
    line-height: 112%;
    color: #f8f6f4;
    padding: 16px;
    margin: -12px 16px 24px 0;
}

.controller-page__info-block:last-child {
    margin-right: 0;
}

.controller-page__info-block p {
    margin: 0 0 12px 0;
}

.controller-page__info-block span {
    font-size: 17px;
}

.controller-page__info-container {
    display: flex;
    justify-content: space-between;
}

.pies-block {
    display: flex;
    justify-content: space-between;
}

.pies-block:first-child {
    margin-bottom: 16px;
}

.dashboard-map div {
    padding: 0;
    height: 346px !important;
}

.dashboard-map p {
    padding: 24px;
    margin: 0;
}

.there-is-data {
    position: absolute;
    display: flex;
    background: #f8f6f4;
    width: 100%;
    height: 100%;
    justify-content: center;
    align-items: center;
    color: #0e1626;
    font-size: 20px;
    font-weight: 500;
    font-size: 14px;
    text-transform: uppercase;
    border-radius: 16px;
}

.dashboard-table {
    position: relative;
}

.dashboard-spectable {
    height: 314px;
}

.dashboard-table__title {
    margin-left: 8px;
}

.info-line__title-errors div:first-child {
    padding-left: 34px;
    justify-content: flex-start;
}

.controller-data__dashboard-errors div {
    justify-content: flex-start;
    padding: 0 6px;
}

.ymaps-2-1-79-map,
.ymaps-2-1-79-inner-panes {
    border-radius: 8px;
}

@media (min-width: 1600px) {
    .controller-data {
        height: 55vh;
    }

    .loader {
        margin-top: 230px;
    }

    .controller-page__info-block {
        font-size: 16px;
    }

    .pie-energy {
        top: 35%;
    }

    .measured-at__dashboard {
        font-size: 11px !important;
    }
}

@media (min-width: 1700px) {
    .measured-at__dashboard {
        font-size: 12px !important;
    }

    .info-block__second div {
        width: 28.5%;
    }

    .pie-energy {
        top: 38%;
    }

    .controller-data__dashboard {
        height: 226px;
    }

    .info-line__title-dashboard {
        font-size: 11px;
    }

    .info-line__table div {
        font-size: 13px;
    }
}

@media (min-width: 1800px) {
    .controller-data {
        height: 60vh;
    }

    .info-line__title-dashboard {
        font-size: 12px;
    }

    .measured-at__dashboard {
        font-size: 12px !important;
    }

    .pie-voltage {
        top: 42% !important;
    }

    .pie-value {
        top: 45%;
    }

    .pie-energy {
        top: 37%;
    }
}

@media (min-width: 1900px) {
    .controller-data {
        height: 63vh;
    }

    .measured-at__dashboard {
        width: 80px !important;
    }

    .info-line__table div {
        font-size: 13px;
    }

    .info-line__title-dashboard {
        font-size: 13px;
    }

    .loader {
        margin-top: 270px;
    }

    .info-block__second div {
        width: 29%;
    }

    .pie-container canvas {
        width: 75% !important;
        height: 75% !important;
    }
}

@media (min-width: 2200px) {
    .pie-container canvas {
        width: 60% !important;
        height: 60% !important;
    }
}

@media (min-width: 2500px) {
    .pie-container canvas {
        width: 50% !important;
        height: 50% !important;
    }
}
</style>