<template>
    <tr className="controller-btn" v-for="controller in controllerList" :key="controller"
        @click="openControllerPage(controller.id)">
        <td class="tr-id">{{ controller.id }}</td>
        <td class="tr-name"> {{ controller.sn }}</td>
        <td>{{ controller.name }}</td>
        <td>{{ controller.status.bat_v }}</td>
        <td>{{ controller.status.dbi }}</td>
        <td>{{ controller.status.created_at }}</td>
        <td>{{ controller.status.event }}</td>
        <td class="tr-btns"></td>
    </tr>
</template>

<script>

export default {
    props: {
        controllerList: Array
    },
    methods: {
        openControllerPage(id) {
            this.$emit('openMainControllerPage', id);
        }
    }
}
</script>

<style>
.controller-btn {
    cursor: pointer;
}
</style>