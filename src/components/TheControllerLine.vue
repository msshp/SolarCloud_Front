<template>
    <tr v-for="controller in controllerList" :key="controller">
        <td class="tr-id">{{ controller.id }}</td>
        <td class="tr-name"> {{ controller.sn }}</td>
        <td>{{ controller.name }}</td>
        <td>{{ controller.model }}</td>
        <td>{{ controller.device_type }}</td>
        <td>{{ controller.status_code }}</td>
        <td class="tr-btns">
            <!-- <button @click="showDeleteWindow(controller.id)" class="basket"></button> -->
            <!-- <button class="settings"></button> -->
        </td>
        <!-- <TheDeleteUser v-if="deleteUserVis" @closeDeleteWindow="closeDeleteWindow" :controllerIdDel="controllerIdDel"
            @deleteUserFromUserList="deleteUserFromUserList" /> -->
    </tr>
</template>

<script>

export default {
    props: {
        controllerList: Array
    }
}
</script>