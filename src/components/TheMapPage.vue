<template>
    <div v-if="map.loading" class="loading">
        <div class="loading-center">
            <div class="loader map-loader"></div>
        </div>
    </div>
    <div v-if="map.map" class="page-content__container-map">
        <div className="map-content map-page">
            <div id="map" style="width: 100%; height: 100%;"></div>
        </div>
        <div v-if="widgetVisibility" class="map-widget">
            <div class="map-widget__padding">
                <div v-if="widget.loading" class="loading widget-loading widget-loader">
                    <div class="loading-center">
                        <div class="loader map-loader"></div>
                    </div>
                </div>
                <div @click="openContrPage" class="map-widget__title">
                    <div class="map-widget__icon"></div>{{ controllerMapInfo.name }}
                </div>
                <div>
                    <h3 class="accordeon-item__title">Информация</h3>
                    <div class="accordeon-item__content accordeon-item__contentainer-info">
                        <div class="accordeon-item__content-info">
                            <div>Cерийный номер</div>
                            <div>Тип контроллера</div>
                            <div>Описание</div>
                            <div>Уровень сигнала GSM</div>
                            <div>Статус нагрузки</div>
                        </div>
                        <div class="accordeon-item__content-info">
                            <div>{{ controllerMapInfo.sn }}</div>
                            <div>{{ controllerMapInfo.device_type.device_type }}</div>
                            <div>{{ controllerMapInfo.device_type.description }}</div>
                            <div>{{ lastDataForWidget.dbi }}</div>
                            <div>{{ lastDataForWidget.load_mode }}</div>
                        </div>
                    </div>
                    <div class="accordeon-item">
                        <h3 class="accordeon-item__title">Последние измерения <div class="lastdate_widget"
                                v-bind:class="{ lastdata_green: lastTimeColor }">{{
                                    lastDataForWidget.measured_at }}</div>
                        </h3>
                    </div>
                    <div class="accordeon-item__content">
                        <div>
                            <p class="slider-title">Уровень заряда АКБ</p>
                            <div class="slider-container"><input id="inputbatc" type="range" class="no-slider" min="0"
                                    max="100" disabled>
                                <div>{{ lastDataForWidget.bat_c }} <span>%</span></div>
                            </div>
                        </div>
                        <div>
                            <p class="slider-title">Напряжение АКБ</p>
                            <div class="slider-container"><input id="inputbatv" type="range" class="no-slider" disabled>
                                <div>{{ lastDataForWidget.bat_v }} <span>В</span></div>
                            </div>
                        </div>
                        <div class="slider-title lastval">
                            <div>Ток АКБ</div>
                            <div>{{ lastDataForWidget.bat_i }} <span>А</span></div>
                        </div>
                        <div class="slider-title lastval">
                            <div>Ток нагрузки</div>
                            <div>{{ lastDataForWidget.load_i }} <span>А</span>
                            </div>
                        </div>

                    </div>
                </div>
                <div>
                    <div>
                        <h3 class="accordeon-item__title">
                            Ошибки</h3>
                    </div>
                    <div class="accordeon-item__content">
                        <div className="info-line info-line__table info-line__widget"
                            v-for="info in lastErrorsForWidget" :key="info">
                            <div class="measured-at__widget-errors">{{ info.measured_at }}</div>
                            <div class="measured-at__dashboard-code measured-at__widget-errval">{{ info.name }}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    props: {
        saveUserData: Object
    },
    data() {
        return {
            allDevicesStorage: [],
            allDevicesInfoPlacemark: [],
            newArrayWithMergedObjects: [],
            widgetVisibility: false,
            placemarks: [],

            controllerMapInfo: {
                account: null,
                created_at: "",
                description: "",
                device_type: { id: null, device_type: '' },
                gps: "",
                id: null,
                installer: null,
                last_session: "",
                name: "",
                pin: null,
                sn: "",
                status: {},
                updated_at: ""
            },
            map: {
                loading: true,
                map: false
            },
            widget: {
                loading: true
            },
            lastDataForWidget: {
                bat_c: "-",
                bat_i: "-",
                bat_v: "-",
                dbi: "-",
                id: null,
                measured_at: "-",
                pv_i: "-",
                pv_v: "-"
            },
            lastErrorsForWidget: [],
            activeId: null,
            lastTimeColor: true
        }
    },
    mounted() {
        axios.get(`http://cloud.io-tech.ru/api/users/${this.saveUserData.id}/devices/?limit=100000`, // запросить координаты
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.allDevicesStorage = response.data.results;
                this.getInfoForPlaceMark();
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    },
    methods: {
        getInfoForPlaceMark() {
            axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000', // запросить статусы
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.allDevicesInfoPlacemark = response.data.results;
                    this.mergePropertiesForPlaceMark(); // объединить координаты и статусы в один объект
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        mergePropertiesForPlaceMark() {
            const mergeArraysById = () => {
                const mergedArray = [];
                this.allDevicesStorage.forEach((deviceStorage) => {
                    const matchingDevice = this.allDevicesInfoPlacemark.find((deviceInfo) => deviceInfo.id === deviceStorage.id);

                    if (matchingDevice) {
                        const mergedDevice = { ...deviceStorage, ...matchingDevice };
                        mergedArray.push(mergedDevice);
                    }
                });
                return mergedArray;
            }

            this.newArrayWithMergedObjects = mergeArraysById();
            this.drawMap(); // нарисовать карту
        },
        drawMap() {
            // карта
            this.map.loading = false;
            this.map.map = true;

            ymaps.ready(() => {
                const myMap = new ymaps.Map('map', {
                    center: [55.76, 37.64],
                    zoom: 10
                });

                // Добавляем обработчик события клика на карту
                myMap.events.add('click', () => {
                    this.widgetVisibility = false;
                    this.placemarks.forEach(function (placemark) {
                        placemark.balloon.close(); // Закрываем балун у каждой метки
                    });
                });

                this.newArrayWithMergedObjects.forEach((el) => {

                    if (el.gps !== null) {
                        let result;
                        let lastCharacter = el.gps.slice(-1);

                        if (lastCharacter === ',') {
                            result = el.gps;
                            const latitude = 55.823966980820266;
                            const longitude = 37.50593900570508;

                            result = { latitude, longitude };

                        } else {
                            result = this.convertCoordinates(el.gps);
                        }

                        // установить цвет метки

                        // Красный – нет связи
                        // Желтый – низкое напряжение АКБ (ниже 11.5В)
                        // Зеленый – все ок

                        let icon = 'greencircle.svg';
                        let thisdate = this.formatDate(el.status.last_session);
                        let targetDate = new Date(thisdate); // Целевая дата
                        let currentDate = new Date(); // Текущее время

                        let diffInHours = Math.abs(targetDate - currentDate) / (1000 * 60 * 60); // Вычисляем разницу в часах между целевой датой и текущим временем

                        if (diffInHours >= 3) { // Проверяем, больше ли разница 3 часов
                            icon = 'redcircle.svg';
                        } else {
                            if (el.status.bat_v !== null && el.status.bat_v < 11.5) {
                                icon = 'yellowcircle.svg';
                            }
                        }

                        let myPlacemark = new ymaps.Placemark([result.latitude, result.longitude], {
                            hintContent: el.name,
                            balloonContent: `<div class="ballon-name">${el.name}</div><div class="ballon-sn">${el.sn}</div>`
                        }, {
                            iconLayout: 'default#image',
                            iconImageHref: `../${icon}`,
                            iconImageSize: [20, 20],
                            iconImageOffset: [-8, -5],
                            hideIconOnBalloonOpen: false
                        });

                        myPlacemark.events.add('click', () => {
                            this.widgetVisibility = false;
                            this.getController(el.id);
                        });
                        myMap.geoObjects.add(myPlacemark);
                        this.placemarks.push(myPlacemark);
                    }
                })
            });
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
        getController(id) {
            this.widget.loading = true;
            this.widgetVisibility = true; // показать виджеты ! без этого не нарисуются шкалы
            this.activeId = id;

            // Информация об устройстве
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.controllerMapInfo = response.data;
                        if (this.controllerMapInfo.device_type.description === undefined) {
                            this.controllerMapInfo.device_type.description = '–';
                        }
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });

            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/fixdata/?date_start=2024-02-01&limit=1`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastDataForWidget = response.data[0];

                        let date = this.lastDataForWidget.measured_at;
                        let formatDate = date.split(',');
                        this.lastDataForWidget.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                        //////// установить цвет плашки с временем
                        this.lastTimeColor = true; // зелёный
                        let thisdate = this.formatDate(date);
                        let targetDate = new Date(thisdate); // Целевая дата
                        let currentDate = new Date(); // Текущее время

                        let diffInHours = Math.abs(targetDate - currentDate) / (1000 * 60 * 60); // Вычисляем разницу в часах между целевой датой и текущим временем

                        if (diffInHours >= 3) { // Проверяем, больше ли разница 3 часов
                            this.lastTimeColor = false;
                        }
                        /////////////

                        setTimeout(() => { // нарисовать значения
                            this.setLastValues();
                        }, 100);
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
            this.getErrorsForWidget(id);
        },
        openContrPage() {
            this.$emit('openMainControllerPage', this.controllerMapInfo.id);
        },
        calculatePercentage(inputVoltage) {
            let minVoltage = 10;
            let maxVoltage = 14.5;
            let percentage = (inputVoltage - minVoltage) / (maxVoltage - minVoltage) * 100;
            return percentage;
        },
        setLastValues() {
            let batCInput = document.getElementById('inputbatc');
            // Установить значение из data() в атрибут value элемента input
            batCInput.value = Number(this.lastDataForWidget.bat_c);

            let colorBatC = '';
            if (batCInput.value < 50) {
                colorBatC = '#E94B4B';
            } else if (batCInput.value > 70) {
                colorBatC = '#B6DE14';
            } else {
                colorBatC = '#F4CA8D';
            }

            batCInput.style.setProperty('background', `-webkit-linear-gradient(left, ${colorBatC} 0%, ${colorBatC} ${batCInput.value}%, #c9c7c5 ${batCInput.value}%, #c9c7c5 100%)`, 'important');

            let batVInput = document.getElementById('inputbatv');
            // 10 - 14.5 вольт
            batVInput.value = this.calculatePercentage(this.lastDataForWidget.bat_v);

            let colorBatV = '';
            if (batVInput.value < 50) {
                colorBatV = '#E94B4B';
            } else if (batVInput.value > 70) {
                colorBatV = '#B6DE14';
            } else {
                colorBatV = '#F4CA8D';
            }

            batVInput.style.setProperty('background', `-webkit-linear-gradient(left, ${colorBatV} 0%, ${colorBatV} ${batVInput.value}%, #c9c7c5 ${batVInput.value}%, #c9c7c5 100%)`, 'important');

            this.widget.loading = false;
        },
        getErrorsForWidget(id) {
            // Ошибки
            this.lastErrorsForWidget = [];
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/event/?type=3&limit=3`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastErrorsForWidget = response.data;

                        if (this.lastErrorsForWidget.length > 3) {
                            this.lastErrorsForWidget = this.lastErrorsForWidget.slice(this.lastErrorsForWidget.length - 3); // вырезаем последние 10 элементов
                        }

                        this.lastErrorsForWidget.forEach((el) => {
                            let date = el.measured_at;
                            let formatDate = date.split(',');
                            el.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                        })
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
        }
    }
};
</script>

<style>
.map-page {
    padding: 0 !important;
    height: 100% !important;
}

.ymaps-2-1-79-map {
    width: 100% !important;
}

.page-content__container-map {
    height: 100%;
    position: relative;
    overflow-y: hidden;
}

.ballon-name {
    background-color: #293B5F;
    color: #f8f6f4;
    text-align: center;
    border-radius: 6px;
}

.ballon-sn {
    height: 20px;
    color: #293B5F;
    font-weight: 500;
}

.map-widget {
    width: 31%;
    height: 100%;
    background-color: #f8f6f4;
    position: absolute;
    top: 0;
    right: 0;
    box-shadow: 0 8px 16px 0 rgba(41, 59, 95, 0.08);
}

.map-widget__padding {
    padding: 80px 24px;
}

.map-widget__title {
    display: flex;
    align-items: center;
    font-weight: 600;
    font-size: 16px;
    line-height: 107%;
    text-transform: uppercase;
    color: #293b5f;
    margin-bottom: 32px;
    cursor: pointer;
}

.map-widget__icon {
    background-image: url(../img/map-widget.svg);
    width: 28px;
    height: 28px;
    margin-right: 8px;
}

.map-widget h3 {
    font-weight: 500;
    font-size: 14px;
    line-height: 122%;
    color: #293b5f;
    text-align: left;
    margin-left: 16px;
    margin-bottom: 16px;
}

.accordeon-item__content {
    display: flex;
    flex-direction: column;
    border-radius: 8px;
    background: #eee;
    padding: 16px;
    margin-bottom: 16px;
}

.accordeon-item__content-info {
    display: flex;
    flex-direction: column;
}

.accordeon-item__content-info:first-child {
    margin-right: 32px;
    font-weight: 500;
    font-size: 13px;
    line-height: 129%;
    color: #293b5f;
}

.accordeon-item__content-info:last-child {
    font-weight: 500;
    font-size: 13px;
    line-height: 129%;
    color: #0e1626;
}

.accordeon-item__content-info div {
    height: 24px;
}

.map-loader {
    margin-top: 0px !important;
    width: 80px !important;
}

.loading-center {
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.accordeon-item__title {
    display: flex;
    font-size: 14px;
    align-items: center;
    margin-left: 16px;
    line-height: 122%;
    color: #293b5f;
}

.lastdate_widget {
    display: flex;
    border-radius: 8px;
    padding: 4px 12px;
    background: #de640c;
    color: #f8f6f4;
    font-weight: 400;
    align-items: center;
    justify-content: center;
    margin-left: 22px;
}

.accordeon-item__contentainer-info {
    flex-direction: row;
}

.no-slider {
    -webkit-appearance: none;
    width: 70%;
    height: 10px;
    border-radius: 5px;
    background: linear-gradient(to right, green 0%, green calc(50% - 1px), #ddd calc(50% + 1px), #ddd 100%);
}

.no-slider::-webkit-slider-thumb {
    display: none;
}

.slider-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 600;
    font-size: 13px;
    line-height: 129%;
    color: #293b5f;
    margin-bottom: 16px;
}

.slider-container div {
    width: 80px;
}

.slider-container span,
.slider-title span {
    color: rgba(0, 0, 0, 0.3);
    width: 40px;
}

.slider-title {
    font-weight: 600;
    font-size: 13px;
    line-height: 129%;
    color: #293b5f;
    margin: 0 0 8px 2px;
}

.lastval {
    display: flex;
}

.lastval div:first-child {
    width: 90px;
}

.lastval div:last-child {
    margin-left: 24px;
}

.info-line__widget {
    justify-content: flex-start !important;
    background-color: transparent !important;
}

.ymaps-2-1-79-balloon {
    border-radius: 6px;
}

.ymaps-2-1-79-balloon__layout {
    border-radius: 6px;
}

.lastdata_green {
    background-color: #9bbd11;
}

.widget-loading {
    top: 0;
    background-color: #f8f6f4 !important;
}

.widget-loader {
    margin-top: -40px !important;
}

.measured-at__widget-errors {
    width: 110px !important;
    font-weight: 500;
    font-size: 13px;
    line-height: 129%;
    color: #293b5f;
    margin-right: 56px;
}

.measured-at__widget-errval {
    font-weight: 500;
    font-size: 13px;
    line-height: 129%;
    color: #0e1626;
    justify-content: flex-start !important;
}

@media (max-width: 1460px) {
    .lastdate_widget {
        margin-left: 8px;
    }
}

@media (max-width: 1600px) {
    .accordeon-item__content-info {
        margin-right: 16px !important;
        font-size: 12px !important;
    }

    .measured-at__widget-errors {
        margin-right: 16px;
    }
}

@media (max-width: 1515px) {

    .accordeon-item__content-info,
    .measured-at__widget-errors,
    .measured-at__dashboard-code {
        font-size: 11px !important;
    }
}
</style>