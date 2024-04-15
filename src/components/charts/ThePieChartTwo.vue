<template>
    <div class="pie-container">
        <canvas id="p_genChart"></canvas>
        <div class="pie-value pie-energy">{{ value }}<p>вт⋅ч</p>
        </div>
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
        let ctx = document.getElementById('p_genChart');

        if (this.controllerInfoStorage.length === 0) {
            this.value = 0;
        } else {
            this.controllerInfoStorage.forEach(el => {
                this.value = this.value + el.p_gen;
            })
        }

        const chartdata = {
            datasets: [{
                data: [this.value],
                borderWidth: [0, 0],
                backgroundColor: [
                    '#2384c5'
                ],
                hoverBackgroundColor: [
                    '#3396D8'
                ],
                hoverOffset: 0
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