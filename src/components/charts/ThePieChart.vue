<template>
    <canvas id="bat_cChart"></canvas>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
    },
    mounted() {
        let ctx = document.getElementById('bat_cChart');

        console.log(this.controllerInfoStorage);

        let labels = [];
        let data = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.measured_at);
        });

        let sum = this.controllerInfoStorage.reduce((sum, current) => sum + current.bat_c, 0);

        data.push(sum);

        new Chart(ctx, {
            type: 'doughnut',
            data: {
                datasets: [
                    {
                        label: 'Уровень заряда акб',
                        data: data,
                    }
                ]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                    },
                    title: {
                        display: true
                    }
                }
            }
        });
    }
}

</script>