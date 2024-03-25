<template>
    <tr v-for="user in usersList" :key="user">
        <td class="tr-id">{{ user.id }}</td>
        <td class="tr-name"> {{ user.first_name + ' ' + user.last_name }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.profile.role }}</td>
        <td>{{ user.profile.created }}</td>
        <td>{{ user.profile.updated }}</td>
        <td class="tr-btns">
            <button @click="showDeleteWindow(user.id)" class="basket"></button>
            <button @click="showEditWindow(user.id)" class="settings"></button>
        </td>
    </tr>
    <TheDeleteUser v-if="deleteUserVis" @closeDeleteWindow="closeDeleteWindow" :userIdDel="userIdDel"
        @deleteUserFromUserList="deleteUserFromUserList" />
    <TheEditUser v-if="editUserVis" @closeEditWindow="closeEditWindow" :userIdDel="userIdDel"
        @editUserFromUserList="editUserFromUserList" :roleChange="roleChange" />
</template>

<script>
// import axios from 'axios';
import TheDeleteUser from './TheDeleteUser.vue'
import TheEditUser from './TheEditUser.vue'

export default {
    components: {
        TheDeleteUser,
        TheEditUser
    },
    data() {
        return {
            deleteUserVis: false,
            editUserVis: false,
            userIdDel: '',
            roleChange: true
        }
    },
    props: {
        usersList: Array
    },
    methods: {
        showDeleteWindow(id) {
            this.deleteUserVis = true;
            this.userIdDel = id;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        showEditWindow(id) {
            this.editUserVis = true;
            this.userIdDel = id;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        closeDeleteWindow(data) {
            this.deleteUserVis = data; // закрытие окна «Добавление пользователя»
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        closeEditWindow(data) {
            this.editUserVis = data; // закрытие окна «Обновление пользователя»
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        deleteUserFromUserList(idUser) {
            // удаление пользователя из списка
            this.$emit('deleteUserFromMainUserList', idUser);
        },
        editUserFromUserList(data) { // обновить информацию о юзере, который отредактировали
            this.$emit('editUserFromMainUserList', data);            
        }
    }
}
</script>


<style>

.window-delete {
    left: 0;
    background-color: rgb(14 22 38 / 25%) !important;
}
</style>
