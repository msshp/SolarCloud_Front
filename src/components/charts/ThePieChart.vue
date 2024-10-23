<template>
    <div class="pie-container">
        <p class="pie-last-time">{{ lastvalueTime }}</p>
        <canvas id="bat_cChart"></canvas>
        <div class="pie-value">{{ value }}%</div>
    </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        telemetryData: Object,
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
        let ctx = document.getElementById('bat_cChart');
        let lastvalue = this.telemetryData;

        if (lastvalue === undefined) {
            this.lastvalueTime = this.lastResult[0].created_at;
            this.value = this.lastResult[0].bat_c;
        } else {

            if (lastvalue.created_at.includes(',')) {
                let formatDate = lastvalue.created_at.split(',');
                lastvalue.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
            }

            this.lastvalueTime = lastvalue.created_at;
            this.value = lastvalue.bat_c;
        }

        let difference = 100 - this.value;

        if (Number(this.value) < 50) {
            this.chartColor = '#E94B4B';
        } else if (Number(this.value) > 70) {
            this.chartColor = '#B6DE14';
        } else {
            this.chartColor = '#F4CA8D';
        }

        if (Number(this.value) === 0) {
            this.chartDefColor = '#E94B4B';
        }

        const chartdata = {
            datasets: [{
                data: [Number(this.value), Number(difference)],
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