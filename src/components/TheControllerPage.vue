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
                    <span>{{ dParameters.type }}</span>
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
                <button @click="dataOn()" v-bind:class="{ controller_btn_active: btns.dataActive }">Детальные
                    данные</button>
                <button @click="eventsOn()" v-bind:class="{ controller_btn_active: btns.eventsActive }">События</button>
                <button @click="commandsOn()"
                    v-bind:class="{ controller_btn_active: btns.commandsActive }">Команды</button>
            </div>
            <div class="data-filter__container">
                <div className="datetext">{{ dateText }}</div>
                <button v-if="btns.dataActive" @click="saveXMLData()" class="account__add-users save-xml__data">Скачать
                    XML</button>
                <button v-if="btns.eventsActive" @click="saveXMLEvents()"
                    class="account__add-users save-xml__data">Скачать XML</button>
                <div class="controller-nav__data-filter">
                    <button class="dropdown__button dropdown__button-data-filter" @click="chooseAccessLevel()"
                        v-bind:class="{ opened_button: selectortimeVisible }">{{ selectortimeContent }}<i
                            class="white_delta"
                            v-bind:class="{ inverted_white_delta: selectortimeVisible }"></i></button>
                    <ul class="dropdown__list dropdown__list-data-filter"
                        v-bind:class="{ dropdown__list_visible: selectortimeVisible }">
                        <div class=" .datafilter-separator datafilter-separator_set"></div>
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
                                <ThePieVoltage :key="forceRenderKey" v-if="batCChart" :telemetryData="telemetryData"
                                    :lastResult="lastResult" />
                            </div>
                        </div>
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Уровень заряда АКБ</h4>
                                <ThePieChart :key="forceRenderKey" v-if="batCChart" :telemetryData="telemetryData"
                                    :lastResult="lastResult" />
                            </div>
                        </div>
                    </div>
                    <div class="pies-block">
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Сгенерированно энергии</h4>
                                <p class="pie-last-time">{{ dateText }}</p>
                                <ThePieChartTwo v-if="visibleChart" :controllerInfoStorage="receivedData"
                                    :energyStorage="energyStorage" />
                            </div>
                        </div>
                        <div class="info-block__half">
                            <div class="info-block__block">
                                <h4 class="charge-level">Потрачено энергии</h4>
                                <p class="pie-last-time">{{ dateText }}</p>
                                <ThePieChartThree v-if="visibleChart" :controllerInfoStorage="receivedData"
                                    :energyStorage="energyStorage" />
                            </div>
                        </div>
                    </div>
                </div>
                <div class="info-block__block info-block__half info-block__errors dashboard-table">
                    <div v-if="thereIsEvents" class="there-is-data">Нет событий за период</div>
                    <div v-if="btns.loadingDashboard" class="loading loading-dashboard_events">
                        <div>
                            <div class="loader"></div>
                        </div>
                    </div>
                    <div>
                        <div class="info-block__errors-container">
                            <h4>события</h4>
                            <button class="save-btn more-btn" @click="eventsOn()">больше →</button>
                        </div>
                        <div className="info-line info-line__title info-line__title-errors">
                            <div class="measured_at measured-at__dashboard-errors">дата/время</div>
                            <div class="error-desc measured-at__dashboard-errors_code">Код</div>
                            <div class="error-desc measured-at__dashboard-name">Описание</div>
                            <div class="error-desc">Тип</div>
                        </div>
                        <div class="controller-data__dashboard-errors">
                            <div className="info-line info-line__table dashboard-event-line"
                                v-for="info in dashboardLastEvents" :key="info"
                                :style="{ backgroundColor: info.color }">
                                <div class="measured_at measured-at__dashboard-errors">{{ info.measured_at }}</div>
                                <div class="measured-at__dashboard-errors_code">{{ info.code }}</div>
                                <div class="measured-at__dashboard-code measured-at__dashboard-name">{{ info.name }}
                                </div>
                                <div class="measured-at__dashboard-code measured-at__dashboard-type">{{ info.type }}
                                </div>
                            </div>
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
                    <div class="info-block__block dashboard-map__container-map">
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
            <div class="controller-info__containerone">
                <div class="controller-info">
                    <div class="title_button">
                        <div class="controller-info__block-title">Данные</div>
                        <div v-if="access" class="controller-info__delete" @click="deleteControllerFromSettings()">
                            <button>Удалить контроллер</button>
                            <div></div>
                        </div>
                    </div>

                    <div class="important_registers">
                        <div>
                            <p class="controller-info__created"><span>ID</span> {{ controllerInfo.id }}</p>
                            <p class="controller-info__created"><span>Hardware version контроллера</span> {{
                                dParameters.hardcontr }}</p>
                            <p class="controller-info__created"><span>Hardware version модема</span> {{
                                dParameters.hardmod }}</p>
                            <p class="controller-info__created"><span>Адрес сервера</span> {{
                                dParameters.serveraddress }}</p>
                        </div>
                        <div>
                            <p class="controller-info__created"><span>Дата создания</span> {{ controllerInfo.created_at
                                }}</p>
                            <p class="controller-info__created"><span>Software version контроллера</span> {{
                                dParameters.cversion }}</p>
                            <p class="controller-info__created"><span>Software version модема</span> {{
                                dParameters.mversion }}</p>
                            <p class="controller-info__created"><span>Порт сервера</span> {{ dParameters.serverport }}
                            </p>
                        </div>
                    </div>

                    <div class="controller-info__name">
                        <p>Название</p>
                        <input type="text" v-model="controllerInfo.name">
                    </div>
                    <div>
                        <p>Описание</p>
                        <input type="text" v-model="controllerInfo.description">
                    </div>
                    <div class="settings-btns_container">
                        <button @click="updateInfo()">Обновить</button>
                        <div class="update" v-bind:class="{ update_visible: updateVisible, update_error: updateError }">
                            {{ updateText
                            }}</div>
                    </div>
                </div>
                <div class="controller-info controller-map">
                    <div id="map-settings" style="width: 100%; height: 82%;"></div>
                    <div class="controller-map__set-coords">
                        <div class="options-block__coords-inpcont">
                            <div>
                                <div class="options-block__coords-inp">
                                    <div @click="switchCoordinates()" class="icon-checkbox-blank"
                                        v-bind:class="{ manual_coords: this.autoSwitchCoords }">
                                    </div>
                                    <div class="options-block__coords-inptitle">Координаты от устройства</div>
                                    <div class="controller-nav__data-filter">
                                        <button
                                            class="dropdown__button dropdown__button-data-filter dropdown__set dropdown__set-coord"
                                            @click="chooseCoords()" v-bind:class="{ opened_button: selectorCoord }">{{
                                                selectorCoordContent
                                            }}<i class="white_delta"
                                                v-bind:class="{ inverted_white_delta: selectorCoord }"></i></button>
                                        <ul class="dropdown__list dropdown__list-data-filter dropdown__list-set"
                                            v-bind:class="{ dropdown__list_visible: selectorCoord }">
                                            <div class="datafilter-separator datafilter-separator_set"></div>
                                            <li v-for="coord in recordedСoordinates" :key="coord"
                                                class="dropdown__list-item" @click="chooseCoordAuto(coord.value)">{{
                                                    coord.value }}</li>
                                        </ul>
                                    </div>
                                </div>
                                <div class="options-block__coords-inp options-block__coords-inp_m">
                                    <div @click="switchCoordinates()" class="icon-checkbox-blank"
                                        v-bind:class="{ manual_coords: this.manualCoords }">
                                    </div>
                                    <div class="options-block__coords-inptitle">Установить вручную</div><input
                                        class="options-block__input options-block__input-manual" id="manual"
                                        v-model.trim="manCoordsInput" v-bind:class="{ manual_color: this.manualColor }"
                                        type="text">
                                    <button class="save-btn save-set" @click="saveNewCoords()">Сохранить</button>
                                </div>
                                <div class="update update-gps"
                                    v-bind:class="{ update_visible: setVisible, update_error: updateError }">{{
                                        setUpdateText }}</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="controller-info controller-info__param">
                <p class="controller-info__block-title controller-info__block-title_par">Параметры</p>
                <div className="info-line info-line__title info-line__par">
                    <div class="num-reg">№ регистра</div>
                    <div class="name-par">Параметр</div>
                    <div class="val-par">Значение</div>
                    <div class="history-par"></div>
                </div>
                <div>
                    <div className="info-line info-line__par_line" v-for="parameter in parametersStorage"
                        :key="parameter">
                        <div class="num-reg">{{ parameter.num }}</div>
                        <div class="name-par">{{ parameter.namepar }}</div>
                        <div class="val-par">{{ parameter.val }}
                        </div>
                        <div class="history-par">{{ parameter.history }}
                        </div>
                    </div>
                </div>
            </div>
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
        <div v-if="btns.eventsActive" class="dashboard-table">
            <div v-if="thereIsEventsTable" class="there-is-data">Нет событий за период</div>
            <div className="info-line info-line__title">
                <div class="measured_at">дата/время</div>
                <div>Код</div>
                <div class="eventstable-desc">Описание</div>
                <div class="eventstable-type">Тип</div>
            </div>
            <div class="controller-data">
                <div className="info-line" v-for="event in lastEvents" :key="event"
                    :style="{ backgroundColor: event.color }">
                    <div class="measured_at">{{ event.measured_at }}</div>
                    <div>{{ event.code }}</div>
                    <div class="eventstable-desc">{{ event.name }}</div>
                    <div class="eventstable-type">{{ event.type }}</div>
                </div>
            </div>
        </div>
        <div v-if="btns.commandsActive" class="dashboard-table">
            <div class="controller-info controller-info__param-commands">
                <div class="controller-info__param-commands-div">
                    <p class="controller-info__block-title controller-info__block-title_par">Добавить команду в очередь
                    </p>
                    <div className="info-line info-line__title info-line__par">
                        <div class="num-reg">№ регистра Параметр</div>
                        <div class="val-par">Текущее значение</div>
                        <div class="val-par">Новое значение</div>
                    </div>
                    <div class="value-string"><button class="dropdown__button dropdown__button-command"
                            @click="chooseContrType()" v-bind:class="{ opened_button: selectorVisible }">{{
                                selectorContent }}<i class="white_delta white_delta-commands"
                                v-bind:class="{ inverted_white_delta: selectorVisible }"></i></button>
                        <ul class="dropdown__list dropdown__list-command"
                            v-bind:class="{ dropdown__list_visible: selectorVisible }">
                            <div class="div-command"></div>
                            <li class="dropdown__list-item dropdown__list-item-command"
                                v-for="parameter in parametersStorage" :key="parameter"
                                @click="chooseThisType(parameter)">
                                <div>{{
                                    parameter.num }}</div> <span>{{ parameter.namepar }}</span>
                            </li>
                        </ul>
                        <div class="current-val">{{ currentValueCommand }}</div>
                        <div class="val-par"><input id="new-val-c" type="text"></div>
                        <div><button class="save-btn save-btn-command" @click="saveNewFlex()">Сохранить</button></div>
                    </div>
                </div>
            </div>
            <div class="controller-info__param-commands-div command-queue">
                <p class="controller-info__block-title controller-info__block-title_par">Очередь команд</p>
                <div className="info-line info-line__title info-line__par info-line-queue">
                    <div class="num-reg-queue">№ регистра</div>
                    <div class="name-par-queue">Параметр</div>
                    <div class="val-par-queue">Отправлено значение</div>
                    <div class="val-par-queue">Дата отправки</div>
                    <div class="response-queue">Ответ</div>
                    <div class="val-par-queue">Дата регистрации ответа</div>
                </div>
                <div>
                    <div className="info-line info-line__par_line info-line-queue" v-for="parameter in commandQueue"
                        :key="parameter">
                        <div class="num-reg-queue">{{ parameter.num }}</div>
                        <div class="name-par-queue">{{ parameter.namepar }}</div>
                        <div class="val-par-queue">{{ parameter.val }}</div>
                        <div class="val-par-queue">{{ parameter.timeSent }}</div>
                        <div class="response-queue">{{ parameter.response }}</div>
                        <div class="val-par-queue">{{ parameter.timeReg }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <TheDeleteController v-if="deleteControllerVis" @closeDeleteWindow="closeDeleteWindow"
        :controllerIdDel="controllerIdDel" @deleteControllerFromList="deleteControllerFromList"></TheDeleteController>
</template>
<script>
import axios from 'axios';
import * as XLSX from 'xlsx'; // Импортируем библиотеку xlsx
import { EventSourcePolyfill } from "event-source-polyfill";

import TheBarChart from './charts/TheBarChart.vue';
import TheVoltageChart from './charts/TheVoltageChart.vue';
import ThePieChart from './charts/ThePieChart.vue';
import ThePieChartTwo from './charts/ThePieChartTwo.vue';
import ThePieChartThree from './charts/ThePieChartThree.vue';
import ThePieVoltage from './charts/ThePieVoltage.vue';
import TheDeleteController from './TheControllerListComponents/TheDeleteController.vue';
// import { getElementsAtEvent } from 'vue-chartjs';

export default {
    components: {
        TheBarChart,
        TheVoltageChart,
        ThePieChart,
        ThePieChartTwo,
        ThePieChartThree,
        ThePieVoltage,
        TheDeleteController
    },
    props: {
        controllerId: Text,
        access: Boolean
    },
    data() {
        return {
            btns: {
                dashBoardActive: true,
                settingsActive: false,
                dataActive: false,
                eventsActive: false,
                commandsActive: false,
                loading: true,
                loadingDashboard: true
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
            thereIsEvents: false,
            thereIsEventsTable: false,

            lastEvents: [],
            dashboardLastEvents: [],

            parametersStorage: [],

            coord: null, // координаты для дашборда
            coordinates: { latitude: 55.76, longitude: 37.64 },
            mapIndication: false,

            manualCoords: false,
            autoSwitchCoords: false,
            coordNow: '',

            manCoordsInput: '',
            newManualCoords: [],
            manualColor: false,
            newCoordSaved: false,

            deleteControllerVis: false,
            updateVisible: false,
            setVisible: false,
            updateError: false,
            setError: false,
            updateText: '',
            setUpdateText: '',

            selectorCoord: false,
            recordedСoordinates: [],
            selectorCoordContent: '–',

            dParameters: {
                type: '',
                cversion: '',
                mversion: '',
                typeconnect: '',
                hardcontr: '',
                hardmod: '',
                serveraddress: '',
                softcontr: '',
                softmod: '',
                serverport: ''
            },

            energyStorage: null, // хранилище энергии (сгенерированно)

            commandQueue: [],

            selectorVisible: false,
            selectorContent: 'Выбрать',
            currentValueCommand: '',
            newParameterData: null,


            // SSE

            telemetryData: "Нет данных", // Данные телеметрии
            eventSource: null, // Источник событий SSE
            reconnectAttempts: 0, // Попытки переподключения
            messageQueue: [], // Очередь для сообщений
            processing: false, // Флаг обработки

            forceRenderKey: 0 // для перерисовки компонентов с диаграмами напряжение и уровень заряда
        }
    },
    methods: {
        chooseContrType() {
            this.selectorVisible = !this.selectorVisible;
        },
        chooseThisType(parameter) {
            this.selectorContent = parameter.num + ' ' + parameter.namepar;
            this.currentValueCommand = parameter.val; // выбор из списка
            this.newParameterData = parameter; // выбор из списка
            this.selectorVisible = false;
        },
        dashBoardOn() {
            if (!this.btns.dashBoardActive) {
                for (let btn in this.btns) { // выключение всех кнопок
                    this.btns[btn] = false;
                }
                this.btns.dashBoardActive = true;
                this.drawControllerMap(this.controllerInfoStorage[0]);
            }
        },
        settingsOn() {
            if (!this.btns.settingsActive) {
                for (let btn in this.btns) { // выключение всех кнопок
                    this.btns[btn] = false;
                }
                this.btns.settingsActive = true;
                this.drawSettingsMap(this.controllerInfoStorage[0]);
            }
        },
        dataOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.dataActive = true;
        },
        eventsOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.eventsActive = true;
        },
        commandsOn() {
            for (let btn in this.btns) { // выключение всех кнопок
                this.btns[btn] = false;
            }
            this.btns.commandsActive = true;
        },
        chooseAccessLevel() {
            this.selectortimeVisible = !this.selectortimeVisible;
        },
        twoDigits(num) {
            return ('0' + num).slice(-2);
        },
        showDay() { //  по умолчанию
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
            this.thereIsEvents = false;
            this.thereIsEventsTable = false;
            this.btns.loadingDashboard = true;

            this.selectortimeVisible = false; // закрыть селектор

            let start = this.dateStart.split('-');
            let end = this.dateEnd.split('-'); // ['2024', '04', '17']

            this.dateText = `${start[2]}.${start[1]} – ${end[2]}.${end[1]}`;
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=${this.dateStart}&limit=100000`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.controllerInfoStorage = response.data;

                        this.energyStorage = this.controllerInfoStorage[0];
                        this.controllerInfoStorage.shift();

                        if (!this.mapIndication) {
                            this.drawControllerMap(this.controllerInfoStorage[0]);
                        }

                        if (this.controllerInfoStorage.length === 0) {
                            this.indicatorSearchLastEntry = true; // найти последнюю запись
                            this.thereIsData = true; // показать блок «Нет данных за этот период»
                            this.receivedData = [];
                        }

                        if (this.controllerInfoStorage.length !== 0) {
                            this.receivedData = this.controllerInfoStorage; //исходный ответ с сервера (для графиков и последнего выхода на связь)
                            let date = this.receivedData[0].created_at;
                            let formatDate = date.split(',');


                            this.receivedData.forEach((el) => {
                                let a = el.created_at.split(',')
                                el.created_at = a[0].slice(0, -5) + ' ' + a[1].slice(0, -3); // дата без вывода секунд
                            })

                            this.telemetryData = this.receivedData[0]; // последние данные (для напряжения и уровня заряда акб)

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
                this.connectSSE(); // подключиться к серверу (SSE)
            }
            this.getErrors();
        },
        connectSSE() { // to do
            if (!sessionStorage.getItem('token')) {
                alert("Ошибка доступа!");
                return;
            }

            this.eventSource = new EventSourcePolyfill(`http://cloud.io-tech.ru/iocloud_sse/`, { // на сервере будет другой адрес iocloud
                // Обязательно передаем токен в заголовке
                headers: {
                    'Token': sessionStorage.getItem('token')
                }
            });

            this.eventSource.onopen = () => {
                this.reconnectAttempts = 0;
                // console.log("Соединение установлено");
            }

            this.eventSource.onmessage = (event) => {
                let messageData = JSON.parse(event.data);

                // Добавляем каждое сообщение в очередь
                this.messageQueue.push(messageData);

                if (!this.processing) {
                    this.processMessages();
                }
            };

            this.eventSource.onerror = () => {
                console.error("Ошибка соединения");
                this.reconnectAttempts += 1;

                if (this.reconnectAttempts > 5) {
                    this.eventSource.close();
                } else {
                    this.eventSource.close();
                    setTimeout(() => this.connectSSE(), 3000); // Пробуем переподключиться через 3 секунды
                }
            };
        },

        async processMessages() {
            this.processing = true;  // Устанавливаем флаг, что идет обработка

            while (this.messageQueue.length > 0) {
                const message = this.messageQueue.shift();  // Забираем первое сообщение из очереди

                try {
                    // Асинхронная обработка сообщения (например, сохранение в локальное хранилище)
                    await this.handleMessage(message);
                } catch (error) {
                    console.error("Ошибка при обработке сообщения:", error);
                }
            }

            this.processing = false;  // Завершаем обработку
        },

        async handleMessage(message) {
            // Пример асинхронной обработки сообщения
            return new Promise((resolve) => {

                // if (message.ping) {
                //     console.log("Пинг:", message);
                // }

                if (message.status) {
                    if (this.controllerInfo.sn === message.status.device_sn) {
                        // console.log(this.controllerInfo.sn)
                        // console.log(message.status.device_sn)


                        this.telemetryData = message.status.data;  // Обновляем последние данные на экране

                        // console.log("Cтатус для устройства (" + message.status.device_sn + ") обновлен!");
                        // console.log("Данные: ", message.status.data);

                        // console.log(this.telemetryData)

                        let date = this.telemetryData.created_at;
                        let formatDate = date.split(',');

                        this.lastReleaseDate = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                        this.reRender(); //  перерисовка компонентов с напряжением и уровнем заряда
                    }
                }

                // if (message.command) {
                //     console.log("Данные команды: ", message.command.data)
                // }

                resolve();
            });
        },
        reRender() {
            this.forceRenderKey++;
        },
        getErrors() {
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/event/?date_start=${this.dateStart}`, // вкладка «События»
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastEvents = response.data;

                        if (this.lastEvents.length === 0) {
                            this.thereIsEventsTable = true; // показать блок «Нет событий за период
                            this.btns.loading = false;
                            this.getEventsForDashbard();
                        } else {
                            this.lastEvents.forEach((el) => {
                                let date = el.measured_at;
                                let formatDate = date.split(',');
                                el.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                                const codeToTypeMap = {
                                    'Info': 'Информация',
                                    'Warning': 'Предупреждение',
                                    'Error': 'Ошибка'
                                };

                                el.type = codeToTypeMap[el.event_type];

                                const colorToTypeMap = {
                                    'Info': '#F8F6F4',
                                    'Warning': '#F4CA8D',
                                    'Error': '#f57878'
                                };

                                el.color = colorToTypeMap[el.event_type];
                            })
                            this.btns.loading = false;
                            this.getEventsForDashbard();
                        }
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        getEventsForDashbard() {
            let currentDate = new Date();

            // Вычитаем две недели (14 дней)
            currentDate.setDate(currentDate.getDate() - 14);

            // Форматируем дату в виде 'гггг-мм-дд'
            let formattedDate = currentDate.toISOString().slice(0, 10);

            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/event/?date_start=${formattedDate}`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.dashboardLastEvents = response.data.slice(0, 17);

                        if (this.dashboardLastEvents.length === 0) {
                            this.thereIsEvents = true; // показать блок «Нет событий за период
                            this.btns.loadingDashboard = false;
                        } else {
                            this.dashboardLastEvents.forEach((el) => {
                                let date = el.measured_at;
                                let formatDate = date.split(',');
                                el.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                                const codeToTypeMap = {
                                    'Info': 'Информация',
                                    'Warning': 'Предупреждение',
                                    'Error': 'Ошибка'
                                };

                                el.type = codeToTypeMap[el.event_type];

                                const colorToTypeMap = {
                                    'Info': '#F8F6F4',
                                    'Warning': '#F4CA8D',
                                    'Error': '#f57878'
                                };

                                el.color = colorToTypeMap[el.event_type];
                            })

                            this.btns.loadingDashboard = false;
                        }
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        drawControllerMap(lastdata) {
            if (!this.newCoordSaved && this.coordNow !== null) {
                this.coord = this.convertCoordinates(this.coordNow);
            }

            if (this.coord) {
                if ((this.coord.latitude !== 0)) {
                    this.coordinates = this.coord;
                    if (this.controllerInfo.coordinates_manually) {
                        this.manCoordsInput = this.coordinates.latitude + ',N,' + this.coordinates.longitude + ',E';
                    }
                }
            }

            let element = document.getElementById('map-dashboard');

            if (element !== null) {
                if (element.firstChild !== null) {
                    element.removeChild(element.firstChild);
                }
            }

            let zoom = 15;
            if (this.coordinates.latitude === 55.76 && this.coordinates.longitude === 37.64) { // если нет координат от устройства, то масштаб оставить 10
                zoom = 10;
            }
            ymaps.ready(() => {
                const dashMap = new ymaps.Map('map-dashboard', {
                    center: [this.coordinates.latitude, this.coordinates.longitude],
                    zoom: zoom
                });
                if (this.coord !== null && this.coord !== undefined) {
                    // установить цвет метки

                    // Красный – нет связи
                    // Желтый – низкое напряжение АКБ (ниже 11.5В)
                    // Зеленый – все ок

                    let icon = 'greencircle.svg';
                    if (lastdata !== undefined) {
                        let thisdate = this.formatDate(lastdata.measured_at);
                        let targetDate = new Date(thisdate); // Целевая дата
                        let currentDate = new Date(); // Текущее время

                        let diffInHours = Math.abs(targetDate - currentDate) / (1000 * 60 * 60); // Вычисляем разницу в часах между целевой датой и текущим временем

                        if (diffInHours >= 3) { // Проверяем, больше ли разница 3 часов
                            icon = 'redcircle.svg';
                        } else {
                            if (lastdata.bat_v !== null && lastdata.bat_v < 11.5) {
                                icon = 'yellowcircle.svg';
                            }
                        }
                    } else {
                        icon = 'redcircle.svg';
                    }

                    let myPlacemark = new ymaps.Placemark([this.coord.latitude, this.coord.longitude], {
                        hintContent: this.controllerInfo.name,
                        balloonContent: `<div class="ballon-name">${this.controllerInfo.name}</div><div class="ballon-sn">${this.controllerInfo.sn}</div>`
                    }, {
                        iconLayout: 'default#image',
                        iconImageHref: `../${icon}`,
                        iconImageSize: [20, 20],
                        iconImageOffset: [-8, -5],
                        hideIconOnBalloonOpen: false
                    });
                    dashMap.geoObjects.add(myPlacemark);

                    this.mapIndication = true;
                }
            });
        },
        drawSettingsMap(lastdata) { // lastdata undefined (если нет данных за период)
            if (this.coordNow !== null) {
                this.coord = this.convertCoordinates(this.coordNow);
            }
            // if (!this.newCoordSaved && this.coordNow !== null) {
            //     this.coord = this.convertCoordinates(this.coordNow);
            // }
            this.manualColor = false; // выкл цвет у инпута с ручными

            if (this.coord) {
                if ((this.coord.latitude !== 0)) {
                    this.coordinates = this.coord;
                    this.selectorCoordContent = `${this.coordinates.latitude}` + ',N,' + `${this.coordinates.longitude}` + ',E'; // по факту какие координаты установлены?
                }
            }

            // let autoSwitchCoords = this.autoSwitchCoords;

            let self = this; // Сохраняем ссылку на this

            let zoom = 15;
            if (this.coordinates.latitude === 55.76 && this.coordinates.longitude === 37.64) { // если нет координат от устройства, то масштаб оставить 10
                zoom = 10;
            }

            ymaps.ready(() => {
                const settingsMap = new ymaps.Map('map-settings', {
                    center: [this.coordinates.latitude, this.coordinates.longitude],
                    zoom: zoom
                }, {
                    cursor: 'pointer'
                });

                if (this.coord !== null && this.coord !== undefined) {
                    // установить цвет метки

                    // Красный – нет связи
                    // Желтый – низкое напряжение АКБ (ниже 11.5В)
                    // Зеленый – все ок

                    let icon = 'greencircle.svg';
                    if (lastdata !== undefined) {
                        let thisdate = this.formatDate(lastdata.measured_at);
                        let targetDate = new Date(thisdate); // Целевая дата
                        let currentDate = new Date(); // Текущее время

                        let diffInHours = Math.abs(targetDate - currentDate) / (1000 * 60 * 60); // Вычисляем разницу в часах между целевой датой и текущим временем

                        if (diffInHours >= 3) { // Проверяем, больше ли разница 3 часов
                            icon = 'redcircle.svg';
                        } else {
                            if (lastdata.bat_v !== null && lastdata.bat_v < 11.5) {
                                icon = 'yellowcircle.svg';
                            }
                        }
                    } else {
                        icon = 'redcircle.svg'; // lastdata undefined (нет данных за период)
                    }

                    let myPlacemark = new ymaps.Placemark([this.coord.latitude, this.coord.longitude], {
                        hintContent: this.controllerInfo.name,
                        balloonContent: `<div class="ballon-name">${this.controllerInfo.name}</div><div class="ballon-sn">${this.controllerInfo.sn}</div>`
                    }, {
                        iconLayout: 'default#image',
                        iconImageHref: `../${icon}`,
                        iconImageSize: [20, 20],
                        iconImageOffset: [-8, -5],
                        hideIconOnBalloonOpen: false
                    });
                    settingsMap.geoObjects.add(myPlacemark);

                    settingsMap.events.add('click', function (e) {

                        if (self.manualCoords) { // если ручные
                            // Код обработчика события
                            let newCoord = e.get('coords');
                            document.getElementById('manual').value = `${newCoord[0]},N,${newCoord[1]},E`;
                            settingsMap.geoObjects.removeAll();

                            let newPlacemark = new ymaps.Placemark([newCoord[0], newCoord[1]], {}, {
                                iconLayout: 'default#image',
                                iconImageHref: `../${icon}`,
                                iconImageSize: [20, 20],
                                iconImageOffset: [-8, -5],
                                hideIconOnBalloonOpen: false
                            });
                            settingsMap.geoObjects.add(newPlacemark);
                        }
                    });
                }
            });
            setTimeout(() => {
                if (!this.autoSwitchCoords) { // если ручные, то вкл курсор
                    this.listenMap();
                } else {
                    this.turnOffMap();
                }
            }, 1000)
        },
        convertCoordinates(coordinates) {
            let lastCharacter = this.coordNow.slice(-1);
            if (lastCharacter === ',') {
                const [lat, long] = this.controllerInfo.gps.split(',');
                return { latitude: lat, longitude: long };
            } else {
                const [lat, dirLat, lon, dirLon] = coordinates.split(',');
                const latitude = [lat, dirLat];
                const longitude = [lon, dirLon];
                return { latitude: latitude[0], longitude: longitude[0] };
            }
        },
        searchLastEntry() {
            axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/fixdata/?date_start=2024-05-01&limit=10000`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastResult = response.data;
                        this.lastResult.shift();
                        // this.lastResult = response.data.shift();

                        if (this.lastResult[0] !== undefined) {
                            let date = this.lastResult[0].created_at;
                            let formatDate = date.split(',');
                            this.lastResult[0].created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                            this.lastReleaseDate = this.lastResult[0].created_at;
                        } else {
                            this.lastResult[0] = {
                                created_at: '-'
                            }
                        }

                        this.batCChart = true;
                        this.lastReleaseSignal = '-';
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        formatDate(date) {
            let dateString = date;
            let [datePart, timePart] = dateString.split(',');

            let [day, month, year] = datePart.split('.');
            let [hours, minutes, seconds] = timePart.split(':');

            let formattedDate = new Date(year, month - 1, day, hours, minutes, seconds);
            return formattedDate.toString();
        },
        switchCoordinates() {
            this.manualCoords = !this.manualCoords;
            this.autoSwitchCoords = !this.autoSwitchCoords;

            if (this.manualCoords) {
                this.listenMap();
            } else {
                this.turnOffMap();
            }

            if (this.manualCoords) {
                this.manCoordsInput = 'Поставьте точку на карте';
            } else {
                axios.post(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/gps/`, // вкл авто
                    {
                        "coordinates_manually": false
                    }, {
                    headers: {
                        'Authorization': `Token ${sessionStorage.getItem('token')}`,
                        'Content-Type': 'application/json; charset=utf-8'
                    }
                }).then((response) => { // обработка ошибок
                    if (response.status === 201) { // данные обновлены
                        this.drawAutoCoord();
                    }
                }).catch((error) => {
                    console.log(error);
                });
            }
        },
        listenMap() {
            this.manualColor = true;
            document.querySelector('.ymaps-2-1-79-events-pane').classList.add('put-label');
        },
        turnOffMap() {
            this.manualColor = false;
            document.querySelector('.ymaps-2-1-79-events-pane').classList.remove('put-label');
        },
        saveNewCoords() { // отправить ручные координаты
            let sendCoord = document.getElementById('manual').value;
            let f = sendCoord.split(',');
            this.coord = { latitude: f[0], longitude: f[1] };
            this.newCoordSaved = true; // индикатор для прорисовки карт с новыми координатами
            this.manualCoords = true;

            axios.post(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/gps/`,
                {
                    "gps": `${sendCoord}`,
                    "coordinates_manually": true
                }, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 201) { // данные обновлены
                    this.setUpdateText = 'Сохранено';
                    this.setVisible = true; // показать «обновлено»
                    setTimeout(() => {
                        this.setVisible = false; // показать «обновлено»
                    }, 5000)
                } else {
                    this.setUpdateText = response.data.detail;
                    this.setError = true; // показать «обновлено»
                    setTimeout(() => {
                        this.setError = false; // показать «обновлено»
                    }, 5000)
                }
            }).catch((error) => {
                this.setUpdateText = error;
                this.setError = true; // показать «обновлено»
                setTimeout(() => {
                    this.setError = false; // показать «обновлено»
                }, 5000)
            });
        },
        drawAutoCoord() {
            if (this.autoSwitchCoords) { // если авто
                this.coordNow = this.selectorCoordContent;
                var element = document.getElementById("map-settings");
                while (element.firstChild) {
                    element.removeChild(element.firstChild);
                }
                this.drawSettingsMap(this.controllerInfoStorage[0]);
            }
        },
        deleteControllerFromSettings() {
            this.deleteControllerVis = true;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        closeDeleteWindow(data) {
            this.deleteControllerVis = data; // закрытие окна «Добавление пользователя»
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        deleteControllerFromList(id) {
            // удаление пользователя из списка
            this.$emit('deleteControllerFromList', id);
        },
        chooseCoords() {
            this.selectorCoord = !this.selectorCoord;
        },
        updateInfo() {
            let obj = {
                "name": this.controllerInfo.name,
                "description": this.controllerInfo.description
            }

            axios.patch(`http://cloud.io-tech.ru/api/devices/${this.controllerInfo.id}/`, obj, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 200) { // данные обновлены
                    this.controllerInfo = response.data;

                    let date = this.controllerInfo.created_at;
                    let formatDate = date.split(',');
                    this.controllerInfo.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                    this.updateText = 'Обновлено';
                    this.updateVisible = true; // показать «обновлено»
                    setTimeout(() => {
                        this.updateVisible = false; // показать «обновлено»
                    }, 5000)
                } else if (response.status === 401 || response.status === 404) {
                    this.updateText = response.data.detail;
                    this.updateError = true; // показать «обновлено»
                    setTimeout(() => {
                        this.updateError = false; // показать «обновлено»
                    }, 5000)
                }
            }).catch((error) => {
                this.updateText = error;
                this.updateError = true; // показать «обновлено»
                setTimeout(() => {
                    this.updateError = false; // показать «обновлено»
                }, 5000)
            });
        },
        chooseCoordAuto(coord) {
            this.selectorCoordContent = coord;
            if (this.autoSwitchCoords) { // отправить в базу, если стоит галочка на авто
                axios.post(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/gps/`,
                    {
                        "gps": `${coord}`,
                        "coordinates_manually": false
                    }, {
                    headers: {
                        'Authorization': `Token ${sessionStorage.getItem('token')}`,
                        'Content-Type': 'application/json; charset=utf-8'
                    }
                }).then((response) => { // обработка ошибок
                    if (response.status === 201) { // данные обновлены
                        this.drawAutoCoord(); // если успешно, переставить метку
                    }
                }).catch((error) => {
                    console.log(error);
                });
            }
        },
        saveXMLData() {
            // Заголовки
            const headers = [
                'дата/время',
                'Напряжение PV(В)',
                'Напряжение АКБ(В)',
                'Напряжение нагрузки(В)',
                'Ток PV(А)',
                'Ток АКБ(А)',
                'Ток нагрузки(А)'
            ];

            // Формируем данные для таблицы
            const rows = this.receivedData.map(info => [
                info.created_at,
                info.pv_v,
                info.bat_v,
                info.load_v,
                info.pv_i,
                info.bat_i,
                info.load_i,
            ]);

            // Объединяем заголовки и строки данных
            const data = [headers, ...rows];

            // Создаем книгу Excel и добавляем туда лист с данными
            const worksheet = XLSX.utils.aoa_to_sheet(data);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, 'Данные');

            // Генерируем файл Excel
            XLSX.writeFile(workbook, `Данные / ${this.controllerInfo.name} / ${this.controllerInfo.sn} / ${this.dateText}.xlsx`);
        },
        saveXMLEvents() {
            // Заголовки
            const headers = [
                'дата/время',
                'Код',
                'Описание',
                'Тип'
            ];
            // Формируем данные для таблицы
            const rows = this.lastEvents.map(info => [
                info.measured_at,
                info.code,
                info.name,
                info.type
            ]);

            // Объединяем заголовки и строки данных
            const data = [headers, ...rows];

            // Создаем книгу Excel и добавляем туда лист с данными
            const worksheet = XLSX.utils.aoa_to_sheet(data);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, 'Данные');

            // Генерируем файл Excel
            XLSX.writeFile(workbook, `События / ${this.controllerInfo.name} / ${this.controllerInfo.sn} / ${this.dateText}.xlsx`);
        },
        saveNewFlex() { // to do

            let newV = document.getElementById('new-val-c').value;

            let command = {
                "cmd_type": "set",
                "pdu": this.newParameterData.num,
                "value": newV
            };

            let timeSent = this.formatDateCommand(new Date());

            axios.post(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/command/`,
                command, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 201) { // данные обновлены

                    let timeReg = this.formatDateCommand(new Date());

                    this.commandQueue.push({
                        num: this.newParameterData.num,
                        namepar: this.newParameterData.namepar,
                        val: newV,
                        timeSent: timeSent,
                        response: response.statusText,
                        timeReg: timeReg
                    })
                }
            }).catch((error) => {
                console.log(error);
            });
        },
        formatDateCommand(dateString) {

            const options = {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                hour12: false,
            };

            const date = new Date(dateString);
            const formattedDate = date.toLocaleString('ru-RU', options)
                .replace(',', '')
                .replace(/(\d{2})\.(\d{2})\.(\d{4})/, '$2.$1.$3');
            return formattedDate;
        },
    },
    mounted() {

        this.controllerIdDel = this.controllerId;
        // Информация об устройстве
        axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                if (response.status === 200) {
                    this.controllerInfo = response.data;

                    let date = this.controllerInfo.created_at;
                    let formatDate = date.split(',');
                    this.controllerInfo.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                    this.coordNow = this.controllerInfo.gps;

                    if (this.controllerInfo.coordinates_manually) {
                        this.manualCoords = true; // ручное
                    } else {
                        this.autoSwitchCoords = true; // авто
                    }
                    this.showDay();
                }
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/gps/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                if (response.status === 200) {
                    this.recordedСoordinates = response.data.slice(0, 3);
                }
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        // Параметры устройства (таблица с номерами регистра)
        axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/flexdata/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                if (response.status === 200) {
                    let arr = response.data;

                    for (let key in arr) {
                        let change = true;

                        if (key === '12' || key === '14' || key === '16') { // неизменяемые поля (тип контроллера и версии)
                            change = false;
                        }

                        const keysToExclude = ['14', '16', '40999', '41027', '41028', '41042', '41043', '41044'];

                        if (!keysToExclude.includes(key)) {  // код для выполнения, если key не содержится в массиве keysToExclude
                            if (arr[key] !== null) {
                                this.parametersStorage.push({
                                    'num': Number(key),
                                    'namepar': arr[key].name,
                                    'val': arr[key].value,
                                    'history': '',
                                    'change': change
                                })
                            } else {
                                this.parametersStorage.push({
                                    'num': Number(key),
                                    'namepar': '',
                                    'val': arr[key],
                                    'history': '',
                                    'change': change
                                });
                            }
                        }

                        if (Number(key) === 12 && arr[key] !== null) {
                            this.dParameters.type = arr[key].value;
                        } else if (Number(key) === 14 && arr[key] !== null) {
                            this.dParameters.cversion = arr[key].value;
                        } else if (Number(key) === 41043 && arr[key] !== null) {
                            this.dParameters.mversion = arr[key].value;
                        } else if (Number(key) === 41042 && arr[key] !== null) {
                            this.dParameters.typeconnect = arr[key].value;
                        } else if (Number(key) === 16 && arr[key] !== null) {
                            this.dParameters.hardcontr = arr[key].value;
                        } else if (Number(key) === 41044 && arr[key] !== null) {
                            this.dParameters.hardmod = arr[key].value;
                        } else if (Number(key) === 41027 && arr[key] !== null) { // Адрес сервера
                            this.dParameters.serveraddress = arr[key].value;
                        } else if (Number(key) === 41028 && arr[key] !== null) { // Адрес сервера
                            this.dParameters.serverport = arr[key].value;
                        }
                    }
                }
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    },
    beforeUnmount() {
        if (this.eventSource) {
            this.eventSource.close();
            // console.log('соединение закрыто');
        }
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

.important_registers {
    display: flex;
}

.important_registers div p:first-child {
    margin: 0;
}

.important_registers div:last-child {
    margin-left: 24px;
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
    align-items: center;
    font-size: 14px;
    background-color: #F8F6F4;
}

.info-line:hover {
    background-color: #e0e4ed;
}

.info-line__title {
    margin-bottom: 4px;
    background-color: transparent;
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
    margin-bottom: 16px;
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
    font-family: 'Inter', sans-serif;
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
    padding: 0 10px;
}

.dropdown__set {
    height: 30px;
    font-size: 13px;
    width: 220px;
    padding: 6px 20px 6px 8px !important;
    font-family: 'Inter', sans-serif;
    margin-left: 0px !important;
}

.dropdown__list-data-filter {
    top: 40px;
}

.datafilter-separator {
    margin: 0px 16px 8px 16px;
}

.datafilter-separator_set {
    margin: 0px;
}

.dropdown__list-set li {
    font-size: 14px;
    margin: 0 !important;
    height: 30px !important;
}

.dropdown__list-data-filter li {
    margin: 0 6px 4px;
}

.dropdown__button-data-filter i {
    width: 16px;
    height: 16px;
}

.dropdown__set i {
    width: 14px;
    height: 14px;
    width: 28px;
    height: 100%;
    position: absolute;
    right: -108px;
    border-radius: 8px;
    background-color: #294b8e;
    padding: 0 7px;
    background-size: auto;
}

.data-filter__container {
    display: flex;
}

.datetext {
    display: flex;
    align-items: center;
    margin-right: 14px;
    color: #294b8e;
    height: 40px;
}

.controller-settings {
    display: flex;
    justify-content: space-between;
}

.controller-info input {
    background-color: transparent;
    color: #0E1626;
    border: 1px solid rgb(41 75 142);
    padding: 6px 8px;
    border-radius: 8px;
    font-size: 13px;
    width: 94%;
    line-height: 112%;
    height: 16px;
}

.controller-info button {
    border-radius: 8px;
    padding: 7px 10px;
    height: 100%;
    background: linear-gradient(90deg, #294b8e 27%, #2384c5 100%);
    width: 120px;
    color: #f8f6f4;
    margin-right: 8px;
}

.controller-info__containerone {
    width: 57%;
}

.controller-info,
.controller-map {
    background-color: #F8F6F4;
    border-radius: 8px;
    padding: 24px;
    font-weight: 400;
    font-size: 14px;
    line-height: 129%;
    color: #0E1626;
}

.controller-info {
    margin-right: 16px;
    position: relative;
    padding: 24px;
}

.controller-map {
    padding: 0;
    margin-top: 16px;
    height: 630px;
}

.controller-nav__btns button {
    background-color: transparent;
}

.controller-data {
    height: 57vh;
    overflow-y: scroll;
    overflow-x: hidden;
    border-radius: 12px;
}

.controller-data .info-line:last-child {
    border-radius: 0 0 12px 12px;
    padding-bottom: 4px;
}

.controller-data .info-line:first-child {
    padding-top: 4px;
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
    padding: 16px 16px 0px 26px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.dashboard-container h4,
.controller-info__block-title {
    font-weight: 500;
    margin: 0;
    font-size: 14px;
    text-transform: uppercase;
    text-align: center;
}

.controller-info__block-title_par {
    margin: 0 !important;
}

.info-line__title-dashboard {
    font-size: 10px;
}

.measured-at__dashboard {
    width: 100px !important;
    font-size: 10px !important;
}

.measured-at__dashboard-errors {
    width: 140px !important;
}

.measured-at__dashboard-code {
    width: 200px !important;
    text-align: left !important;
    height: 24px !important;
}

.measured-at__dashboard-type {
    width: 172px !important;
}

.info-line__title-errors {
    font-size: 13px !important;
    font-weight: 500;
    justify-content: flex-start;
    padding-left: 28px;
    width: 92%;
    padding-top: 4px;
    margin-bottom: 0px;
}

.info-line__title-errors div {
    justify-content: flex-start;
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
    padding: 0 0 0 10px;
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
    margin: -12px 16px 16px 0;
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
}

.dashboard-map__container-map {
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
    border-radius: 4px;
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
    width: 146px !important;
}

.controller-data__dashboard-errors div {
    justify-content: flex-start;
    padding: 0 6px;
    text-align: left;
    align-items: center;
}

.ymaps-2-1-79-map,
.ymaps-2-1-79-inner-panes {
    border-radius: 8px;
}

.error-desc {
    padding: 0 6px;
    width: 200px !important;
}

.dashboard-event-line {
    width: 96%;
}

.dashboard-event-line div {
    height: 32px !important;
}

.dashboard-event-line:first-child {
    border-radius: 8px 8px 0 0;
}

.dashboard-event-line:last-child {
    border-radius: 0 0 8px 8px;
}

.measured-at__dashboard-errors_code {
    width: 40px !important;
}

.eventstable-desc {
    width: 480px !important;
    justify-content: flex-start !important;
}

.eventstable-type {
    width: 350px !important;
    justify-content: flex-start !important;
}

.loading-dashboard_events {
    height: 92%;
    margin-top: 50px;
    background-color: #f8f6f4;
    z-index: 90;
    border-radius: 24px;
}

/* карта в настройках (менять координаты) */

.controller-map__set-coords {
    padding: 0px 12px;
    margin-top: 20px;
}

.options-block__coords-inpcont {
    display: flex;
}

.options-block__coords-inp {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
}

.options-block__coords-inp_m {
    margin-bottom: 0px;
}

.options-block__coords-btn {
    display: flex;
    flex-direction: column;
}

.options-block__coords-btn button {
    border-radius: 8px;
    margin-bottom: 8px;
    padding: 6px 10px;
    margin-left: 8px;
    height: 100%;
}

.options-block__coords-inptitle {
    font-weight: 500;
    font-size: 12px;
    line-height: 129%;
    color: #0E1626;
    width: 163px;
    margin-right: 8px;
}

.options-block__input {
    background-color: transparent;
    color: #0E1626;
    border: 1px solid rgba(14, 22, 38, 0.5);
    padding: 6px 8px;
    border-radius: 8px;
    font-size: 13px;
    width: 240px;
    line-height: 112%;
    height: 16px;
}

.manual_color {
    color: #0E1626 !important;
}

.options-block__coords-check {
    margin-top: 6px;
    font-weight: 500;
    font-size: 14px;
    line-height: 129%;
    color: #0E1626;
    display: flex;
    align-items: center;
}

.icon-checkbox-blank {
    background-image: url(../checkbox/blank.svg);
    width: 20px;
    height: 20px;
    background-size: cover;
    background-repeat: no-repeat;
    margin-right: 8px;
    cursor: pointer;
}

.manual_coords {
    background-image: url(../checkbox/done.svg);
}

.put-label {
    cursor: pointer !important;
}

.title_button {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 16px;
}

.controller-info__delete {
    display: flex;
    align-items: center;
    background: #294b8e;
    border-radius: 8px;
    padding: 5px 10px 5px 16px;
    width: 173px;
    height: 24px;
    font-size: 13px;
}

.controller-info__delete button {
    color: #F8F6F4;
    background: #294b8e !important;
    font-size: 14px;
    padding: 0;
    margin: 0;
    width: 168px !important;
    font-family: 'Inter', sans-serif;
}

.controller-info__delete div {
    width: 20px;
    height: 20px;
    background-size: contain;
    background-image: url(../img/account/basket-white.svg);
    background-repeat: no-repeat;
    margin-left: 10px;
    cursor: pointer;
}

.controller-info__name p {
    margin-top: 0;
}

.controller-info__created span {
    color: #294b8e;
}

.update {
    display: none;
    align-items: center;
    justify-content: center;
}

.update-gps {
    font-size: 13px;
    margin-top: 4px;
    margin-right: 21px;
    justify-content: flex-end;
}

.update_error {
    display: block;
    color: #F21616;
}

.update_visible {
    display: flex;
    color: #86a312;
}

.save-set {
    border-radius: 8px;
    padding: 7px 14px;
    margin-left: 8px !important;
    font-family: 'Inter', sans-serif;
}

.save-btn-command {
    margin: 0 0 0 24px !important;
    width: 110px;
}

.dropdown__list-set {
    top: 29px;
    padding-top: 0px;
    padding-bottom: 2px;
    width: 316px;
}

.controller-info__block-title {
    text-align: left;
}

.controller-info__param {
    width: 40%;
    margin: 0;
    padding: 24px 16px;
}

.options-block__input-manual {
    width: 170px !important;
}

.dropdown__set-coord {
    width: 316px !important;
    background: #294b8e !important;
}

.info-line__par {
    padding: 8px 0 0 0;
    font-size: 13px !important;
    justify-content: flex-start;
}

.num-reg {
    width: 36% !important;
}

.history-par {
    width: 1px !important;
}

.name-par {
    width: 96% !important;
}

.val-par {
    width: 29% !important;
    margin-left: 8px;
}

.info-line__par_line div {
    text-align: left;
    justify-content: flex-start;
    font-size: 13px;
    padding: 4px;
}

.info-line__par div {
    text-align: left;
    justify-content: flex-start;
}

.settings-btns_container {
    display: flex;
    margin-top: 16px;
}

.controller-info__param-commands {
    width: 100%;
    padding: 0;
}

.controller-info__param-commands-div {
    padding: 24px 20px;
}

.save-title {
    width: 203px;
}

.dropdown__list-command {
    height: 300px;
    overflow-y: scroll;
    margin: 70px 24px 0px 24px;
    width: 520px;
}

.dropdown__button-command {
    margin: 0 !important;
    width: 520px;
}

.div-command {
    margin: 4px 0;
}

.dropdown__list-item-command {
    margin: 0;
    border-radius: 4px;
}

.dropdown__list-item-command div {
    width: 80px;
    border: none;
    margin: 0;
}

.dropdown__list-item-command span {
    margin-left: 8px;
}

.value-string {
    display: flex;
    align-items: center;
}

.current-val {
    margin-left: 32px;
    width: 164px;
}

.white_delta-commands {
    width: 16px !important;
    height: 16px !important;
}

.command-queue {
    margin-top: 24px;
    background-color: #f8f6f4;
    border-radius: 8px;
}

.num-reg-queue {
    width: 96px !important;
    padding: 0 0 0 8px !important;
}

.name-par-queue {
    width: 380px !important;
    padding: 0 !important;
}

.info-line-queue {
    justify-content: space-between;
}

.val-par-queue {
    width: 185px !important;
    padding: 0 !important;
}

.response-queue {
    width: 140px !important;
    padding: 0px !important;
}

@media (max-width: 1600px) {

    .options-block__coords-btn button,
    .options-block__coords-check {
        font-size: 12px;
    }
}

@media (min-width: 1500px) {
    .options-block__coords-inptitle {
        font-size: 13px;
        width: 180px;
    }

    .info-line__par_line div {
        font-size: 14px;
    }

    .important_registers div:last-child {
        margin-left: 48px;
    }
}

@media (min-width: 1600px) {
    .controller-data {
        height: 63vh;
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

    .dashboard-event-line div {
        height: 34px !important;
    }

    .controller-data__dashboard-errors {
        padding: 0 0 0 12px;
    }

    .options-block__coords-inptitle {
        font-size: 14px;
        width: 193px;
    }

    .dropdown__set-coord {
        width: 316px;
    }

    .dropdown__set i {
        right: -105px;
    }

    .save-set {
        padding: 7px 16px !important;
    }

    .dropdown__list-set {
        width: 316px;
    }

    .options-block__input-manual {
        width: 187px !important;
    }

    .important_registers div:last-child {
        margin-left: 56px;
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

    .info-line__title-errors div:first-child {
        width: 134px !important;
    }

    .dashboard-event-line div {
        height: 39px !important;
    }

    .important_registers div:last-child {
        margin-left: 64px;
    }
}

@media (min-width: 1800px) {
    .controller-data {
        height: 67vh;
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

    .measured-at__dashboard-name {
        width: 250px !important;
    }

    .important_registers div:last-child {
        margin-left: 88px;
    }
}

@media (min-width: 1900px) {
    .controller-data {
        height: 69vh;
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

    .measured-at__dashboard-name {
        width: 370px !important;
    }

    .info-line__table div:first-child {
        font-size: 12px;
    }

    .info-line__title-errors div:first-child {
        width: 110px !important;
    }

    .dashboard-event-line div {
        height: 35px !important;
    }

    .important_registers div:last-child {
        margin-left: 108px;
    }
}

@media (min-width: 2000px) {
    .controller-data {
        height: 70vh;
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

@media (max-width: 1515px) {

    .measured-at__widget-errors,
    .measured-at__dashboard-code {
        font-size: 11px !important;
    }
}

@media (min-width: 1700px) {
    .measured-at__widget-errors {
        width: 116px !important;
        margin-right: 32px;
    }

    .measured-at__dashboard-code {
        width: 215px !important;
        height: 24px !important;
    }
}

@media (min-width: 1800px) {
    .measured-at__dashboard-code {
        width: 250px !important;
    }
}

@media (min-width: 1900px) {
    .measured-at__dashboard-code {
        width: 350px !important;
    }

    .measured-at__widget-errors {
        width: 142px !important;
    }
}

@media (min-width: 2500px) {
    .dashboard-event-line div {
        height: 32px !important;
    }
}
</style>