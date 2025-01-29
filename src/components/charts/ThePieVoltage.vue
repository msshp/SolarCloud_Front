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
        telemetryData: Object,
        lastResult: Array,
        voltageValueSetSystem: Object
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
        let lastvalue = this.telemetryData;

        if (lastvalue === 'Нет данных') {
            lastvalue = undefined;
        }

        if (lastvalue === undefined) {
            this.lastvalueTime = this.lastResult[0].created_at; // последнее пришедшее значение (может быть давно)
            this.value = this.lastResult[0].bat_v;
        } else {
            if (lastvalue.created_at.includes(',')) {
                let formatDate = lastvalue.created_at.split(',');
                lastvalue.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
            }
            this.lastvalueTime = lastvalue.created_at; // последнее пришедшее значение за заданный период
            this.value = lastvalue.bat_v;
        }

        // из 57347 посмотреть систему
        // мин 57358 - макс 57352

        let minVal = this.voltageValueSetSystem.min;
        let maxVal = this.voltageValueSetSystem.max;

        let b = ((this.value - minVal) / (maxVal - minVal)) * 100; // процентное соотношение третьего значения от разницы между максимальным и минимальным.
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