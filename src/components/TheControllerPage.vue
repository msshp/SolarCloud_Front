<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>{{ controllerInfo.name }}</p>
            <div></div>
        </div>
        <nav className="controller-nav">
            <div>
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
        <div class="dashboard-container" v-if="btns.dashBoardActive">
            <div class="info-block__first">
                <div class="info-block__half">
                    <div class="info-block__block info-block__block-current">
                        <TheBarChart v-if="visibleChart" :controllerInfoStorage="controllerInfoStorage" />
                    </div>
                    <div class="info-block__block">
                        <TheVoltageChart v-if="visibleChart" :controllerInfoStorage="controllerInfoStorage" />
                    </div>
                </div>
                <div class="info-block__block info-block__half">
                    <div class="info-block__half-title">
                        <h4>Последние данные</h4><button class="save-btn more-btn" @click="dataOn()">больше →</button>
                    </div>
                    <div className="info-line info-line__title info-line__title-dashboard">
                        <div class="measured_at measured-at__dashboard">дата/время</div>
                        <div>Напряжение PV</div>
                        <div>Напряжение АКБ</div>
                        <div>Напряжение нагрузки</div>
                        <div>Ток PV</div>
                        <div>Ток АКБ</div>
                        <div>Ток нагрузки</div>
                    </div>
                    <div class="controller-data controller-data__dashboard">
                        <div className="info-line info-line__table" v-for="info in smallControllerInfoStorage"
                            :key="info">
                            <div class="measured_at measured-at__dashboard">{{ info.measured_at }}</div>
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
            <div class="info-block__first info-block__second">
                <div class="info-block__block">
                    <h4>Уровень заряда АКБ</h4>
                    <ThePieChart v-if="visibleChart" :controllerInfoStorage="controllerInfoStorage" />
                </div>
                <div class="info-block__block">
                    <h4>Сгенерированная мощность</h4>
                    <ThePieChartTwo v-if="visibleChart" :controllerInfoStorage="controllerInfoStorage" />
                </div>
                <div class="info-block__block">
                    <h4>Потреблённая мощность</h4>
                    <ThePieChartThree v-if="visibleChart" :controllerInfoStorage="controllerInfoStorage" />
                </div>
            </div>
            <div class="info-block__block info-block__errors">
                <h4>ошибки</h4>
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
                <p><span>Модель</span> {{ controllerInfo.model.model }}</p>
                <p><span>Дата создания</span> {{ controllerInfo.created_at }}</p>
            </div>
            <div class="controller-map">Карта</div>
        </div>
        <div v-if="btns.dataActive">
            <div className="info-line info-line__title">
                <div class="measured_at">дата/время</div>
                <div>Напряжение PV</div>
                <div>Напряжение АКБ</div>
                <div>Напряжение нагрузки</div>
                <div>Ток PV</div>
                <div>Ток АКБ</div>
                <div>Ток нагрузки</div>
            </div>
            <div class="controller-data">
                <div className="info-line" v-for="info in controllerInfoStorage" :key="info">
                    <div class="measured_at">{{ info.measured_at }}</div>
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

export default {
    components: {
        TheBarChart,
        TheVoltageChart,
        ThePieChart,
        ThePieChartTwo,
        ThePieChartThree
    },
    props: {
        controllerId: Text
    },
    data() {
        return {
            btns: {
                dashBoardActive: true,
                settingsActive: false,
                dataActive: false
            },
            selectortimeContent: '',
            selectortimeVisible: false,
            visibleChart: false,

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
                model: { id: '', model: '' },
                name: "Устройство",
                pin: '',
                sn: "",
                status: {},
                updated_at: ""
            },

            controllerInfoStorage: [], // данные за период
            smallControllerInfoStorage: []
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
            let datestart = new Date(today.getTime() - (24 * 60 * 60 * 1000));
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
            this.visibleChart = false; // скрыть графики

            this.selectortimeVisible = false; // закрыть селектор
            this.dateText = `${this.dateStart} – ${this.dateEnd}`;
            console.log(this.controllerId);

            // &date_end=${this.dateEnd}
            console.log(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=${this.dateStart}&limit=100000`);
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=${this.dateStart}&limit=100000`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        console.log(response.data.results);
                        this.controllerInfoStorage = response.data.results;
                        // удаление запятой в датах
                        this.controllerInfoStorage.forEach((el) => {
                            el.measured_at = el.measured_at.replace(',', ' ');
                        });

                        this.correctControllerData();
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        correctControllerData() {

            console.log(this.dateEnd);
            // let unformattedLastTime = new Date(parseInt(this.dateEnd.getTime() / 300000) * 300000); // последняя отметка времени, кратная 5
            // let timeInterval = Math.floor((unformattedLastTime - unformattedFirstTime) / (1000 * 60)); // заданный период в минутах (1440 по умолчанию)
            // let numberOfRecords = timeInterval / 5 + 1; // сколько в интервале отметок, кратных 5? (289 по умолчанию)

            this.visibleChart = true;
            this.smallControllerInfoStorage = this.controllerInfoStorage.slice(0, 17); // заполнение таблицы для дашборда
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
                    console.log(response.data)
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

.info-line div {
    text-align: center;
    width: 160px;
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
    color: #0e1626;
    margin-right: 32px;
}

.controller_btn_active {
    font-weight: 700 !important;
}

.controller-nav__data-filter {
    width: 208px;
    position: relative;
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

.controller-data {
    height: 62vh;
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
    padding: 32px;
    font-weight: 400;
    font-size: 14px;
    color: #0E1626;
}

.info-block__half {
    width: 50%;
}

.info-block__half:last-child {
    margin-left: 24px;
    padding: 24px 24px 16px;
}

.info-block__block-current {
    margin-bottom: 24px;
    padding: 24px;
}

.info-block__errors {
    padding: 24px;
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
    width: 230px !important;
    font-size: 10px !important;
}

.info-line__table div {
    font-size: 12px;
}

.controller-data__dashboard {
    overflow: hidden;
    height: 415px !important;
}

.info-block__second div {
    width: 30%;
    margin-right: 24px;
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
}

@media (min-width: 1600px) {
    .controller-data {
        height: 65vh;
    }

    .measured-at__dashboard {
        width: 200px !important;
    }
}

@media (min-width: 1700px) {
    .measured-at__dashboard {
        font-size: 11px !important;
    }
}

@media (min-width: 1800px) {
    .controller-data {
        height: 68vh;
    }

    .info-line__title-dashboard {
        font-size: 12px;
    }

    .measured-at__dashboard {
        font-size: 12px !important;
    }
}

@media (min-width: 1900px) {
    .controller-data {
        height: 71vh;
    }

    .measured-at__dashboard {
        width: 180px !important;
    }

    .info-line__table div {
        font-size: 13px;
    }

    .info-line__title-dashboard {
        font-size: 13px;
    }
}
</style>