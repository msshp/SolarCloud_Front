<template>
    <div class="page-content__container-map">
        <div className="map-content map-page">
            <div id="map" style="width: 100%; height: 100%;"></div>
        </div>
        <div v-if="widgetVisibility" class="map-widget">
            <div class="map-widget__title">
                <div class="map-widget__icon"></div>{{ controllerMapInfo.name }}
            </div>
            <div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(0)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 0) ? '600' : '500' }">
                            Информация</h3>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 0">
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
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 1) ? '600' : '500' }">
                            Последние
                            измерения</h3>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 1">
                    </div>
                </div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(2)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 2) ? '600' : '500' }">
                            Нагрузка</h3>
                    </div>
                    <div class="accordeon-item__content" v-if="activeIndex === 2">
                    </div>
                </div>
                <div>
                    <div class="accordeon-item" @click="toggleAccordion(3)">
                        <h3 class="accordeon-item__title" :style="{ fontWeight: (activeIndex === 3) ? '600' : '500' }">
                            Сеансы связи
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
            lastDataFWidget: {},

            // аккордеон
            activeIndex: 0, // открывает первый элемент по умолчанию
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
            }
        }
    },
    mounted() {
        axios.get(`http://cloud.io-tech.ru/api/users/${this.saveUserData.id}/devices/`,
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
                        // console.log(this.controllerMapInfo);
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
            this.widgetVisibility = true;
            this.getLastDataFWidget(id);
        },
        toggleAccordion(index) {
            this.activeIndex = this.activeIndex === index ? null : index;

            // Установить жирность для всех заголовков
            let headers = document.querySelectorAll('.accordeon-item__title');
            headers.forEach((header, i) => {
                header.style.fontWeight = (i === this.activeIndex) ? '600' : '500';
            });
        },
        getLastDataFWidget(id) {
            axios.get(`http://cloud.io-tech.ru/api/devices/${id}/fixdata/`,
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    if (response.status === 200) {
                        this.lastDataFWidget = response.data[0];
                    }
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                })
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
</style>