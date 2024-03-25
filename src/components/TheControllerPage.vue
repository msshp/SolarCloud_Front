<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Контроллер</p>
            <div></div>
        </div>
        <nav className="controller-nav">
            <div><button @click="dashBoardOn()" v-bind:class="{ controller_btn_active: btns.dashBoardActive }">
                    Дашборд</button>
                <button @click="settingsOn()" v-bind:class="{ controller_btn_active: btns.settingsActive }">
                    Настройки</button>
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
        <div v-if="btns.dashBoardActive">Дашборд</div>
        <div v-if="btns.settingsActive">Настройки</div>
        <div v-if="btns.dataActive">
            <div className=" info-line">
                <div class="measured_at">дата/время</div>
                <div>Напряжение PV</div>
                <div>Напряжение АКБ</div>
                <div>Напряжение нагрузки</div>
                <div>Ток PV</div>
                <div>Ток АКБ</div>
                <div>Ток нагрузки</div>
            </div>
            <div className="info-line" v-for="   info    in    controllerInfoLine   " :key="info">
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
</template>

<script>
import axios from 'axios';

export default {
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

            // даты
            dateEnd: '',
            dateStart: '',
            dateText: '',

            controllerInfoLine: [
                {
                    "id": 1,
                    "measured_at": "17.01.2024,09:42:54",
                    "created_at": "17.01.2024,09:42:54",
                    "pv_v": "18.50",
                    "pv_i": "0.30",
                    "bat_v": "11.50",
                    "bat_i": "0.00",
                    "bat_c": 21,
                    "load_v": "0.00",
                    "load_i": "0.00",
                    "load_st": "0.00",
                    "load_mode": "16",
                    "crg_mode": "2",
                    "temp": "22;25",
                    "p_gen": 11,
                    "p_con": 0,
                    "dbi": "OK",
                    "fault": "1",
                    "fault_modem": "0",
                    "device": 1
                },
                {
                    "id": 1,
                    "measured_at": "17.01.2024,09:42:54",
                    "created_at": "17.01.2024,09:42:54",
                    "pv_v": "18.50",
                    "pv_i": "0.30",
                    "bat_v": "11.50",
                    "bat_i": "0.00",
                    "bat_c": 21,
                    "load_v": "0.00",
                    "load_i": "0.00",
                    "load_st": "0.00",
                    "load_mode": "16",
                    "crg_mode": "2",
                    "temp": "22;25",
                    "p_gen": 11,
                    "p_con": 0,
                    "dbi": "OK",
                    "fault": "1",
                    "fault_modem": "0",
                    "device": 1
                },
                {
                    "id": 1,
                    "measured_at": "17.01.2024,09:42:54",
                    "created_at": "17.01.2024,09:42:54",
                    "pv_v": "18.50",
                    "pv_i": "0.30",
                    "bat_v": "11.50",
                    "bat_i": "0.00",
                    "bat_c": 21,
                    "load_v": "0.00",
                    "load_i": "0.00",
                    "load_st": "0.00",
                    "load_mode": "16",
                    "crg_mode": "2",
                    "temp": "22;25",
                    "p_gen": 11,
                    "p_con": 0,
                    "dbi": "OK",
                    "fault": "1",
                    "fault_modem": "0",
                    "device": 1
                }
            ]
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
            this.selectortimeVisible = false; // закрыть селектор
            this.dateText = `${this.dateStart} – ${this.dateEnd}`;
            console.log(this.controllerId);
            // controllerInfoLine заполнить массив объектами

            console.log(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=${this.dateStart}&date_end=${this.dateEnd}`)
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) { console.log(response.data); }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        }
    },
    mounted() {
        this.showDay();
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

.info-line:first-child {
    margin-bottom: 16px;
}

.info-line div {
    text-align: center;
    width: 160px;
}

.measured_at {
    width: 150px !important;
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
</style>