<template>
    <canvas id="myChart"></canvas>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
    },
    mounted() {
        let ctx = document.getElementById('myChart');

        console.log(this.controllerInfoStorage);

        let labels = [];
        let data = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.measured_at);
            data.push(el.bat_i);
        });

        new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Ток',
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