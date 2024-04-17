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

        let labels = [];
        let dataPvV = [];
        let dataBatV = [];
        let dataLoadV = [];

        this.controllerInfoStorage.forEach((el) => {
            labels.push(el.created_at);
            dataPvV.push(el.pv_v);
            dataBatV.push(el.bat_v);
            dataLoadV.push(el.load_v);
        });

        labels.reverse();
        dataPvV.reverse();
        dataBatV.reverse();
        dataLoadV.reverse();

        new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Напряжение PV (В)',
                        data: dataPvV,
                        borderWidth: 2,
                        borderColor: '#F4CA8D',
                        pointBorderWidth: 0,
                        pointHitRadius: 0,
                        cubicInterpolationMode: 'monotone',
                        pointStyle: 'cross',
                        backgroundColor: '#F4CA8D',
                    },
                    {
                        label: 'Напряжение АКБ (В) / Напряжение нагрузки (В)',
                        data: dataBatV,
                        borderWidth: 2,
                        borderColor: '#B6DE14',
                        pointBorderWidth: 0,
                        pointHitRadius: 0,
                        cubicInterpolationMode: 'monotone',
                        pointStyle: 'cross',
                        backgroundColor: '#B6DE14',
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