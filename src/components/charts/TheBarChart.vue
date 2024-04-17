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
        // заполнение массивов для отображения (3 параметра в одном графике)
        let ctx = document.getElementById('myChart');

        let labels = [];
        let dataBatI = [];
        let dataPvI = [];
        let dataLoadI = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.created_at);
            dataBatI.push(el.bat_i);
            dataPvI.push(el.pv_i);
            dataLoadI.push(el.load_i);
        });

        labels.reverse();
        dataBatI.reverse();
        dataPvI.reverse();
        dataLoadI.reverse();

        new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Ток PV (А)',
                        data: dataPvI,
                        borderWidth: 2,
                        borderColor: '#293B5F',
                        pointBorderWidth: 0,
                        pointHitRadius: 0,
                        cubicInterpolationMode: 'monotone',
                        pointStyle: 'cross',
                        backgroundColor: '#293B5F',
                    },
                    {
                        label: 'Ток АКБ (А)',
                        data: dataBatI,
                        borderWidth: 2,
                        borderColor: '#2384c5',
                        pointBorderWidth: 0,
                        pointHitRadius: 0,
                        cubicInterpolationMode: 'monotone',
                        pointStyle: 'cross',
                        backgroundColor: '#2384c5',
                    },
                    {
                        label: 'Ток нагрузки (А)',
                        data: dataLoadI,
                        borderWidth: 2,
                        borderColor: '#B743C1',
                        pointBorderWidth: 0,
                        pointHitRadius: 0,
                        cubicInterpolationMode: 'monotone',
                        pointStyle: 'cross',
                        backgroundColor: '#B743C1',
                    }
                ]
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