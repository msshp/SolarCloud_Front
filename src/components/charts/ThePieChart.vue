<template>
    <p class="pie-last-time">{{ lastvalueTime }}</p>
    <canvas id="bat_cChart"></canvas>
    <div class="pie-value">{{ value }}%</div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
    },
    data() {
        return {
            value: null,
            chartColor: '',
            chartDefColor: '#A9B8D7',
            lastvalueTime: ''
        }
    },
    methods: {
        twoDigits(num) {
            return ('0' + num).slice(-2);
        }
    },
    mounted() {
        let ctx = document.getElementById('bat_cChart');

        let lastvalue = this.controllerInfoStorage[0];

        let time = new Date(lastvalue.measured_at); // вывести время последней записи
        this.lastvalueTime = this.twoDigits(time.getMonth() + 1) + '/' + this.twoDigits(time.getDate()) + ' ' + this.twoDigits(time.getHours()) + ':' + this.twoDigits(time.getMinutes());

        this.value = lastvalue.bat_c;
        let difference = 100 - lastvalue.bat_c;

        if (Number(lastvalue.bat_c) < 50) {
            this.chartColor = '#E94B4B';
        } else if (Number(lastvalue.bat_c) > 70) {
            this.chartColor = '#B6DE14';
        } else {
            this.chartColor = '#F4CA8D';
        }

        if (Number(lastvalue.bat_c) === 0) {
            this.chartDefColor = '#E94B4B';
        }

        const chartdata = {
            datasets: [{
                data: [Number(lastvalue.bat_c), Number(difference)],
                borderWidth: [0, 0],
                backgroundColor: [
                    this.chartColor,
                    this.chartDefColor
                ],
                hoverBackgroundColor: [
                    this.chartColor,
                    this.chartDefColor
                ],
                hoverOffset: [3, 0]
            }]
        }

        new Chart(ctx, {
            type: 'doughnut',
            data: chartdata,
            options: {
                cutout: 100,
                plugins: {
                    tooltip: {
                        enabled: false
                    }
                }
            }
        });
    }
}

</script>