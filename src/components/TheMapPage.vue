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
            <div @click="openContrPage" class="map-widget__title">
                <div class="map-widget__icon"></div>{{ controllerMapInfo.name }}
            </div>
            <div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(0)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 0) ? '600' : '500' }">
                            Информация</h3>
                    </div>
                    <div class="accordeon-item__content accordeon-item__contentainer-info" v-if="activeIndex === 0">
                        <div class="accordeon-item__content-info">
                            <div>Cерийный номер</div>
                            <div>Тип контроллера</div>
                        </div>
                        <div class="accordeon-item__content-info">
                            <div>{{ controllerMapInfo.sn }}</div>
                            <div>{{ controllerMapInfo.device_type.device_type }}</div>
                        </div>
                    </div>
                </div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(1)">
                        <div class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 1) ? '600' : '500' }">
                            Последние измерения <div class="lastdate_widget">{{ lastDataForWidget.measured_at }}</div>
                        </div>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 1">
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
                                <div>{{ lastDataForWidget.bat_v }} <span>Вольт</span></div>
                            </div>
                        </div>
                        <div>
                            <p class="slider-title">Напряжение PV</p>
                            <div class="slider-container"><input id="inputpvv" type="range" class="no-slider" disabled>
                                <div>{{ lastDataForWidget.pv_v }} <span>Вольт</span></div>
                            </div>
                        </div>
                        <div class="slider-title lastval">
                            <div>Ток PV</div>
                            <div>{{ lastDataForWidget.pv_i }} <span>А</span></div>
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
                    <div class="accordeon-item" @click="toggleAccordion(2)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 2) ? '600' : '500' }">
                            Ошибки</h3>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 2">
                        <div className="info-line info-line__title info-line__widget">
                            <div class="measured_at measured-at__dashboard-errors">дата/время</div>
                            <div>Описание</div>
                        </div>
                        <div>
                            <div className="info-line info-line__table info-line__widget"
                                v-for="info in lastErrorsForWidget" :key="info">
                                <div class="measured_at measured-at__dashboard-errors">{{ info.measured_at }}</div>
                                <div class="measured-at__dashboard-code">{{ info.name }}</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(3)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 3) ? '600' : '500' }">
                            Команды
                        </h3>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 3">
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
            widgetVisibility: false,
            placemarks: [],

            // аккордеон
            activeIndex: 1, // открывает первый элемент по умолчанию
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
        }
    },
    mounted() {
        axios.get(`http://cloud.io-tech.ru/api/users/${this.saveUserData.id}/devices/?limit=100000`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.allDevicesStorage = response.data.results;
                this.drawMap();
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    },
    methods: {
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

                this.allDevicesStorage.forEach((el) => {
                    if (el.gps !== null) {
                        const result = this.convertCoordinates(el.gps);

                        let myPlacemark = new ymaps.Placemark([result.latitude, result.longitude], {
                            hintContent: el.name,
                            balloonContent: `<div class="ballon-name">${el.name}</div><div class="ballon-sn">${el.sn}</div>`
                        });

                        myPlacemark.events.add('click', () => {
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
            // Информация об устройстве
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.controllerMapInfo = response.data;
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });

            // Последние измерения to do
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/fixdata/?date_start=2024-02-01&limit=1`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastDataForWidget = response.data[0];

                        let date = this.lastDataForWidget.measured_at;
                        let formatDate = date.split(',');
                        this.lastDataForWidget.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                        this.widgetVisibility = true; // показать виджеты
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });

            // Ошибки
            this.lastErrorsForWidget = [];
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/event/?type=3&limit=10`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastErrorsForWidget = response.data;
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

            // if (index === 1) { // установить значения для слайдеров если было нажатие на «Последние измерения»
            setTimeout(() => {
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

                let pvVInput = document.getElementById('inputpvv');
                pvVInput.value = this.calculatePercentage(this.lastDataForWidget.pv_v);

                let colorPvV = '';
                if (pvVInput.value < 50) {
                    colorPvV = '#E94B4B';
                } else if (pvVInput.value > 70) {
                    colorPvV = '#B6DE14';
                } else {
                    colorPvV = '#F4CA8D';
                }

                pvVInput.style.setProperty('background', `-webkit-linear-gradient(left, ${colorPvV} 0%, ${colorPvV} ${pvVInput.value}%, #c9c7c5 ${pvVInput.value}%, #c9c7c5 100%)`, 'important');
            }, 1000);
            // }
        },
        toggleAccordion(index) {
            this.activeIndex = this.activeIndex === index ? null : index;

            // Установить жирность для всех заголовков
            let headers = document.querySelectorAll('.accordeon-item__title');
            headers.forEach((header, i) => {
                header.style.fontWeight = (i === this.activeIndex) ? '600' : '500';
            });

            if (index === 1) { // установить значения для слайдеров если было нажатие на «Последние измерения»
                setTimeout(() => {
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

                    let pvVInput = document.getElementById('inputpvv');
                    pvVInput.value = this.calculatePercentage(this.lastDataForWidget.pv_v);

                    let colorPvV = '';
                    if (pvVInput.value < 50) {
                        colorPvV = '#E94B4B';
                    } else if (pvVInput.value > 70) {
                        colorPvV = '#B6DE14';
                    } else {
                        colorPvV = '#F4CA8D';
                    }

                    pvVInput.style.setProperty('background', `-webkit-linear-gradient(left, ${colorPvV} 0%, ${colorPvV} ${pvVInput.value}%, #c9c7c5 ${pvVInput.value}%, #c9c7c5 100%)`, 'important');
                }, 1000);
            }

            if (index === 2) { // обновить ошибки
                this.lastErrorsForWidget = [];
                axios.get(`http://cloud.io-tech.ru/api/devices/${id}/event/?type=3&limit=10`,
                    {
                        headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                    }).then((response) => {
                        if (response.status === 200) {
                            this.lastErrorsForWidget = response.data;
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
            }
        },
        openContrPage() {
            this.$emit('openMainControllerPage', this.controllerMapInfo.id);
        },
        calculatePercentage(inputVoltage) {
            let minVoltage = 10;
            let maxVoltage = 14.5;
            let percentage = (inputVoltage - minVoltage) / (maxVoltage - minVoltage) * 100;
            return percentage;
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
    padding: 56px 0px 0px 0px;
    height: 100%;
    position: relative;
}

.ballon-name {
    background-color: #2680c3;
    color: #f8f6f4;
    text-align: center;
    border-radius: 24px;
}

.ballon-sn {
    height: 20px;
}

.map-widget {
    width: 31%;
    height: 86%;
    background-color: #f8f6f4;
    position: absolute;
    top: 0;
    right: 0;
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

.accordeon-item {
    margin-bottom: 24px;
}

.map-widget h3 {
    font-weight: 500;
    font-size: 14px;
    line-height: 122%;
    color: #293b5f;
    text-align: left;
    margin-left: 16px;
}

.accordeon-item__content {
    display: flex;
    flex-direction: column;
    border-radius: 8px;
    background: #eee;
    padding: 16px;
    margin-bottom: 24px;
}

.accordeon-item__content-info {
    display: flex;
    flex-direction: column;
}

.accordeon-item__content-info:first-child {
    margin-right: 18px;
    font-weight: 500;
    font-size: 13px;
    line-height: 129%;
    color: #293b5f;
}

.accordeon-item__content-info:last-child {
    font-weight: 600;
    font-size: 13px;
    line-height: 129%;
    color: #0e1626;
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
    cursor: pointer;
}

.lastdate_widget {
    display: flex;
    border-radius: 8px;
    padding: 4px 12px;
    background: #de640c;
    color: #f8f6f4;
    font-weight: 300;
    align-items: center;
    justify-content: center;
    margin-left: 16px;
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
}
</style>