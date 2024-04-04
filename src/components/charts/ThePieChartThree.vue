<template>
    <canvas id="p_conChart"></canvas>
    <div class="pie-value pie-energy">{{ value }}<p>кВт⋅ч</p>
    </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        controllerInfoStorage: Array,
    },
    data() {
        return {
            value: null
        }
    },
    mounted() {
        let ctx = document.getElementById('p_conChart');

        this.controllerInfoStorage.forEach(el => {
            this.value = this.value + el.p_con;
        })

        const chartdata = {
            datasets: [{
                data: [this.value],
                borderWidth: [0, 0],
                backgroundColor: [
                    '#293B5F'
                ],
                hoverBackgroundColor: [
                    '#1E2C46'
                ],
                hoverOffset: 0
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