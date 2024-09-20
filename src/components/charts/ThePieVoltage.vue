<template>
    <div class="pie-container">
        <p class="pie-last-time">{{ lastvalueTime }}</p>
        <canvas id="pieVoltage"></canvas>
        <div class="pie-value pie-voltage">{{ value }}<p>Вольт</p>
        </div>
    </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
        lastResult: Array
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
        let ctx = document.getElementById('pieVoltage');
        let lastvalue = this.controllerInfoStorage[0];

        if (lastvalue === undefined) {
            this.lastvalueTime = this.lastResult[0].created_at;
            this.value = this.lastResult[0].bat_v;
        } else {
            this.lastvalueTime = lastvalue.created_at;
            this.value = lastvalue.bat_v;
        }

        // 10 - 14.5 вольт
        let a = this.value - 10;
        let b = (a * 100) / 4.5;
        let c = 100 - b;

        if (b < 50) {
            this.chartColor = '#E94B4B';
        } else if (b > 70) {
            this.chartColor = '#B6DE14';
        } else {
            this.chartColor = '#F4CA8D';
        }

        if (Number(this.value) === 0) {
            this.chartDefColor = '#E94B4B';
        }

        const chartdata = {
            datasets: [{
                data: [b, c],
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
                cutout: 90,
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