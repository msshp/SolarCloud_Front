<template>
    <canvas id="voltageChart"></canvas>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
    },
    mounted() {
        let ctx = document.getElementById('voltageChart');

        console.log(this.controllerInfoStorage);

        let labels = [];
        let data = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.measured_at);
            data.push(el.bat_v);
        });

        new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Напряжение',
                    data: data,
                    borderWidth: 1
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });
    }
}

</script>