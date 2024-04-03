<template>
    <canvas id="bat_cChart"></canvas>
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
            chartColor: ''
        }
    },
    mounted() {
        let ctx = document.getElementById('bat_cChart');

        let labels = [];
        let data = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.measured_at);
        });

        let lastvalue = this.controllerInfoStorage[0];
        this.value = lastvalue.bat_c;
        console.log(lastvalue);
        let difference = 100 - lastvalue.bat_c;

        data.push(lastvalue.bat_c);

        if (Number(lastvalue.bat_c) < 50) {
            this.chartColor = '#E94B4B';
        } else if (Number(lastvalue.bat_c) > 70) {
            this.chartColor = '#B6DE14';
        } else {
            this.chartColor = '#F4CA8D';
        }

        const chartdata = {
            datasets: [{
                data: [Number(lastvalue.bat_c), Number(difference)],
                borderWidth: [0, 0],
                borderColor: '#00CA8B',
                borderRadius: 0,
                backgroundColor: [
                    this.chartColor,
                    '#A9B8D7',
                ],
                hoverOffset: 4
            }]
        };

        new Chart(ctx, {
            type: 'doughnut',
            data: chartdata,
            options: {
                cutout: 80,
            }
        });
    }
}

</script>