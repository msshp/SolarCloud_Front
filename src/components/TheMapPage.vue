<template>
    <div class="page-content__container-map">
        <div className="map-content map-page">
            <div id="map" style="width: 100%; height: 100%;"></div>
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
            allDevicesStorage: []
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
                
                this.allDevicesStorage.forEach((el) => {
                    if (el.gps !== null) {
                        const result = this.convertCoordinates(el.gps);

                        let myPlacemark = new ymaps.Placemark([result.latitude, result.longitude], {
                            hintContent: el.name,
                            balloonContent: `<div class="ballon-name">${el.name}</div><div class="ballon-sn">${el.sn}</div>`
                        });
                        myMap.geoObjects.add(myPlacemark);
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
</style>