<template>
    <div class="pie-container">
        <canvas id="p_genChart"></canvas>
        <div class="pie-value pie-energy">{{ value }}<p>{{ unitsOfMeasurement }}</p>
        </div>
    </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
    props: {
        //controllerInfoStorage: Array,
        energyStorage: Object
    },
    data() {
        return {
            value: null,
            unitsOfMeasurement: 'Вт⋅ч'
        }
    },
    methods: {
        convertWattsToKWh(watts) {
            if (typeof watts === 'number' && watts >= 1000) {
                const kilowatts = watts / 1000;
                this.unitsOfMeasurement = 'кВт⋅ч';
                return kilowatts.toFixed(1);
            } else {
                this.unitsOfMeasurement = 'Вт⋅ч';
                return watts;
            }
        }
    },
    mounted() {
        let ctx = document.getElementById('p_genChart');
        this.value = this.convertWattsToKWh(this.energyStorage.total_p_gen);

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