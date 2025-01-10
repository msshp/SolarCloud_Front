<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Контрольная панель</p>
            <div></div>
        </div>
        <div class="control-page__container">
            <div class="info-block__block control-diagram">
                <h4>Пользователи</h4>
                <div>
                    <canvas id="usersChart"></canvas>
                </div>
            </div>
            <div class="info-block__block control-diagram">
                <h4>Статус контроллеров</h4>
                <div>
                    <canvas id="controllersChart"></canvas>
                </div>
            </div>
            <!-- <div class="info-block__block control-diagram">
                <h4>Общая выработка энергии</h4>
                <div>
                    <canvas id="pgenChart"></canvas>
                </div>
            </div> -->
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
    data() {
        return {
            controlUserList: [],
            currentDate: '',
            pgenStorage: []
        }
    },
    methods: {
        checkConnection(date) {
            let thisdate = this.formatDate(date);
            let targetDate = new Date(thisdate); // Целевая дата

            let diffInHours = Math.abs(targetDate - this.currentDate) / (1000 * 60 * 60); // Вычисляем разницу в часах между целевой датой и текущим временем

            if (diffInHours >= 3) { // Проверяем, больше ли разница 3 часов
                return false;
            } else {
                return true;
            }
        },
        formatDate(date) {
            let dateString = date;
            let [datePart, timePart] = dateString.split(',');

            let [day, month, year] = datePart.split('.');
            let [hours, minutes, seconds] = timePart.split(':');

            let formattedDate = new Date(year, month - 1, day, hours, minutes, seconds);

            return formattedDate.toString();
        }
    },
    mounted() {
        // пользователи
        axios.get('http://cloud.io-tech.ru/api/users/?limit=10000',
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.controlUserList = response.data.results;

                let admins = this.controlUserList.filter(user => user.profile.role === 'Admin').length;
                let moderators = this.controlUserList.filter(user => user.profile.role === 'Moderator').length;
                let regularUsers = this.controlUserList.filter(user => user.profile.role === 'User').length;

                let usersC = document.getElementById('usersChart');

                const usersdata = {
                    labels: [`Админ (${admins})`, `Модератор (${moderators})`, `Наблюдатель (${regularUsers})`],
                    datasets: [{
                        data: [admins, moderators, regularUsers],
                        borderWidth: [0, 0, 0],
                        backgroundColor: [
                            '#293B5F',
                            '#86A5E3',
                            '#8DE386'
                        ],
                        hoverBackgroundColor: [
                            '#293B5F',
                            '#86A5E3',
                            '#8DE386'
                        ]
                    }]
                }

                new Chart(usersC, {
                    type: 'doughnut',
                    data: usersdata,
                    options: {
                        plugins: {
                            tooltip: {
                                enabled: true
                            },
                            legend: {
                                labels: {
                                    font: {
                                        size: 12
                                    },
                                    color: '#293B5F',
                                    padding: 16
                                },
                                position: 'right'
                            }
                        }
                    }
                });

            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        // статус контроллеров
        axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.controlList = response.data.results;
                // ок
                // разряд аккум
                // нет связи 

                let okStorage = [];
                // let dischargedStorage = []; // на связь выходил менее 3 часов назад и батарея разряжена
                let noСonnectionStorage = [];
                this.currentDate = new Date(); // Текущее время

                this.controlList.forEach(element => {
                    if (element.status === null) {
                        noСonnectionStorage.push(element);
                    } else {
                        let connection = this.checkConnection(element.status.created_at);
                        // let battery = this.checkBattery(element.status.bat_v)

                        if (connection) { // на связь выходил менее 3 часов назад и батарея в норме
                            okStorage.push(element);
                        } else { // на связь выходил больше 3 часов назад
                            noСonnectionStorage.push(element);
                        }
                    }
                });

                let contrC = document.getElementById('controllersChart');

                const statusdata = {
                    labels: [`OK (${okStorage.length})`, `Нет связи (${noСonnectionStorage.length})`],
                    datasets: [{
                        data: [okStorage.length, noСonnectionStorage.length],
                        borderWidth: [0, 0],
                        backgroundColor: [
                            '#B6DE14',
                            '#E94B4B'
                        ],
                        hoverBackgroundColor: [
                            '#B6DE14',
                            '#E94B4B'
                        ]
                    }]
                }

                new Chart(contrC, {
                    type: 'doughnut',
                    data: statusdata,
                    options: {
                        plugins: {
                            tooltip: {
                                enabled: true
                            },
                            legend: {
                                labels: {
                                    font: {
                                        size: 12
                                    },
                                    color: '#293B5F',
                                    padding: 16
                                },
                                position: 'right'
                            }
                        }
                    }
                });

            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        // // статус контроллеров
        // axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
        //     {
        //         headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
        //     }).then((response) => {
        //         // обработка успешного запроса
        //         this.pgenStorage = response.data.results;

        //         let pgenC = document.getElementById('pgenChart');

        //         const labels = [1, 2, 3, 4, 5, 6, 7];
        //         const data = {
        //             labels: labels,
        //             datasets: [{
        //                 label: 'My First Dataset',
        //                 data: [65, 59, 80, 81, 56, 55, 40],
        //                 backgroundColor: [
        //                     'rgba(255, 99, 132, 0.2)',
        //                     'rgba(255, 159, 64, 0.2)',
        //                     'rgba(255, 205, 86, 0.2)',
        //                     'rgba(75, 192, 192, 0.2)',
        //                     'rgba(54, 162, 235, 0.2)',
        //                     'rgba(153, 102, 255, 0.2)',
        //                     'rgba(201, 203, 207, 0.2)'
        //                 ],
        //                 borderColor: [
        //                     'rgb(255, 99, 132)',
        //                     'rgb(255, 159, 64)',
        //                     'rgb(255, 205, 86)',
        //                     'rgb(75, 192, 192)',
        //                     'rgb(54, 162, 235)',
        //                     'rgb(153, 102, 255)',
        //                     'rgb(201, 203, 207)'
        //                 ],
        //                 borderWidth: 1
        //             }]
        //         };

        //         new Chart(pgenC, {
        //             type: 'bar',
        //             data: data,
        //             options: {
        //                 scales: {
        //                     y: {
        //                         beginAtZero: true
        //                     }
        //                 },
        //                 plugins: {
        //                     tooltip: {
        //                         enabled: true
        //                     },
        //                     legend: {
        //                         labels: {
        //                             font: {
        //                                 size: 16
        //                             },
        //                             color: '#293B5F',
        //                             padding: 24
        //                         },
        //                         position: 'right'
        //                     }
        //                 }
        //             },
        //         });
        //     }).catch((error) => {
        //         // обработка ошибки
        //         console.log(error);
        //     });
    }
}

</script>

<style>
.control-page__container {
    display: flex;
    justify-content: space-between;
}

.control-diagram {
    width: 45%;
    /* width: 28%;  */
    padding: 24px !important;
}

.control-diagram div {
    display: flex;
    justify-content: center;
    width: 508px;
    height: 508px;
}

.control-diagram h4 {
    font-weight: 500;
    margin: 0;
    font-size: 14px;
    text-transform: uppercase;
    text-align: center;
    margin-bottom: 0;
}

.control-diagram canvas {
    width: 100% !important;
    height: 100% !important;
}
</style>