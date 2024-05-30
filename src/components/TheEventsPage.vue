<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>События</p>
            <div></div>
        </div>
        <div class="dashboard-table"><!--v-if-->
            <div class="info-line info-line__title">
                <div class="measured_at">дата/время</div>
                <div>Код</div>
                <div class="eventstable-desc">Описание</div>
                <div class="eventstable-type">Тип</div>
            </div>
            <div class="controller-data">
                <div class="info-line" style="background-color: rgb(245, 120, 120);">
                    <div class="measured_at">30.05.2024 13:59</div>
                    <div>906</div>
                    <div class="eventstable-desc">Потеря связи с контроллером</div>
                    <div class="eventstable-type">Ошибка</div>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- <script>

import axios from 'axios';

export default {
    data() {
        return {
        }
    },
    mounted() {
        // // Информация об устройстве
        // axios.get(`http://cloud.io-tech.ru/api/devices/${this.controllerId}/`,
        //     {
        //         headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
        //     }).then((response) => {
        //         if (response.status === 200) {
        //             this.controllerInfo = response.data;
        //             this.showDay();
        //         }
        //     }).catch((error) => {
        //         // обработка ошибки
        //         console.log(error);
        //     });
    }
}

</script> -->