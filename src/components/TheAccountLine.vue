<template>
    <tr v-for="user in usersList" :key="user">
        <td class="tr-id">{{ user.id }}</td>
        <td class="tr-name"> {{ user.first_name + ' ' + user.last_name }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.profile.role }}</td>
        <td>{{ user.profile.created }}</td>
        <td>{{ user.profile.updated }}</td>
        <td class="tr-btns"><button @click="showDeleteWindow(user.id)" class="basket"></button>
            <!-- <button class="settings"></button> -->
        </td>
    </tr>
    <TheDeleteUser v-if="deleteUserVis" @closeDeleteWindow="closeDeleteWindow" :userIdDel="userIdDel"
        @deleteUserFromUserList="deleteUserFromUserList" />
</template>

<script>
// import axios from 'axios';
import TheDeleteUser from './TheDeleteUser.vue'

export default {
    components: {
        TheDeleteUser
    },
    data() {
        return {
            deleteUserVis: false,
            userIdDel: ''
        }
    },
    props: {
        usersList: Array
    },
    methods: {
        showDeleteWindow(id) {
            this.deleteUserVis = true;
            this.userIdDel = id;
        },
        closeDeleteWindow(data) {
            this.deleteUserVis = data; // закрытие окна «Добавление пользователя»
        },
        deleteUserFromUserList(idUser) {
            // удаление пользователя из списка
            this.$emit('deleteUserFromMainUserList', idUser);
        }
    }
}
</script>


<style>
tbody tr {
    height: 72px;
}
</style>
