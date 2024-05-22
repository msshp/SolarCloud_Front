<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Контрольная панель</p>
            <div></div>
        </div>
        <div class="control-page__container">
            <div class="info-block__block control-diagram">
                <h4 class="charge-level">Пользователи</h4>
                <div class="pie-container">
                    <canvas id="usersChart"></canvas>
                </div>
            </div>
            <!-- <div class="info-block__block control-diagram">
                <h4 class="charge-level">Статус контроллеров</h4>
                <div class="pie-container">
                    <canvas id="controllersChart"></canvas>
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
            controlUserList: []
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

                const chartdata = {
                    labels: ['Админ', 'Модератор', 'Наблюдатель'],
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
                    data: chartdata,
                    options: {
                        cutout: 120,
                        plugins: {
                            tooltip: {
                                enabled: true
                            }
                        }
                    }
                });

            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        // статус контроллеров
        // axios.get('http://cloud.io-tech.ru/api/users/?limit=10000',
        // {
        //     headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
        // }).then((response) => {
        //     // обработка успешного запроса
        //     this.controlUserList = response.data.results;
        //     console.log(this.controlUserList);

        //     let admins = this.controlUserList.filter(user => user.profile.role === 'Admin').length;
        //     let moderators = this.controlUserList.filter(user => user.profile.role === 'Moderator').length;
        //     let regularUsers = this.controlUserList.filter(user => user.profile.role === 'User').length;

        //     console.log(admins, moderators, regularUsers);

        //     let usersC = document.getElementById('controllersChart');

        //     const chartdata = {
        //         labels: ['Админ', 'Модератор', 'Наблюдатель'],
        //         datasets: [{
        //             data: [admins, moderators, regularUsers],
        //             borderWidth: [0, 0, 0],
        //             backgroundColor: [
        //                 '#293B5F',
        //                 '#86A5E3',
        //                 '#8DE386'
        //             ],
        //             hoverBackgroundColor: [
        //                 '#293B5F',
        //                 '#86A5E3',
        //                 '#8DE386'
        //             ]
        //         }]
        //     }

        //     new Chart(usersC, {
        //         type: 'doughnut',
        //         data: chartdata,
        //         options: {
        //             cutout: 90,
        //             plugins: {
        //                 tooltip: {
        //                     enabled: true
        //                 }
        //             }
        //         }
        //     });

        // }).catch((error) => {
        //     // обработка ошибки
        //     console.log(error);
        // });

    }
}

</script>

<style>
.control-page__container {
    display: flex;
    justify-content: space-between;
}

.control-diagram {
    width: 46%;
}
</style>