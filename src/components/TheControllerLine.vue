<template>
    <tr className="controller-btn" v-for="controller in controllerList" :key="controller">
        <td @click="openControllerPage(controller.id)" class="tr-id">{{ controller.id }}</td>
        <td @click="openControllerPage(controller.id)" class="tr-name"> {{ controller.sn }}</td>
        <td @click="openControllerPage(controller.id)">{{ controller.name }}</td>
        <td @click="openControllerPage(controller.id)">{{ controller.status.bat_v }}</td>
        <td @click="openControllerPage(controller.id)">{{ controller.status.dbi }}</td>
        <td @click="openControllerPage(controller.id)">{{ controller.status.created_at }}</td>
        <td @click="openControllerPage(controller.id)">{{ controller.status.event }}</td>
        <td v-if="access" class="controller-delete__container" @click="deleteController(controller.id)">
            <div class="controller-delete"></div>
        </td>
    </tr>
    <TheDeleteController v-if="deleteControllerVis" @closeDeleteWindow="closeDeleteWindow"
        :controllerIdDel="controllerIdDel" @deleteControllerFromList="deleteControllerFromList" />
</template>

<script>

import TheDeleteController from './TheControllerListComponents/TheDeleteController.vue'

export default {
    components: {
        TheDeleteController
    },
    data() {
        return {
            deleteControllerVis: false,
            controllerIdDel: '',
        }
    },
    props: {
        controllerList: Array,
        access: Boolean
    },
    methods: {
        openControllerPage(id) {
            this.$emit('openMainControllerPage', id);
        },
        deleteController(id) {
            this.deleteControllerVis = true;
            this.controllerIdDel = id;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        deleteControllerFromList(id) {
            // удаление пользователя из списка
            this.$emit('deleteControllerFromList', id);
        },
        closeDeleteWindow(data) {
            this.deleteControllerVis = data; // закрытие окна «Добавление пользователя»
            document.getElementById('page-content').classList.remove('overflow_hidden');
        }
    }
}
</script>

<style>
.controller-btn {
    cursor: pointer;
}

.controller-delete__container {
    border-radius: 0 8px 8px 0;
    padding-right: 12px !important;
}

.controller-delete {
    background-image: url(../../img/account/basket.svg);
    padding: 0 20px 0 0 !important;
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
    height: 30px;
}
</style>