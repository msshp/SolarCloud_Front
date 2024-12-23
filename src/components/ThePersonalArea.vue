<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Личный кабинет</p>
            <div></div>
        </div>
        <table class="pa_page-table">
            <thead>
                <tr class="pa__titles">
                    <th class="tr-id">ID</th>
                    <th class="tr-pa">Имя</th>
                    <th class="tr-pa">email</th>
                    <th>роль</th>
                    <th class="tr-pa">дата создания</th>
                    <th class="tr-pa">дата обновления</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                <tr class="pa__line">
                    <td class="tr-id">{{ saveUserData.id }}</td>
                    <td class="tr-pa"> {{ saveUserData.first_name + ' ' + saveUserData.last_name }}</td>
                    <td class="tr-pa">{{ saveUserData.email }}</td>
                    <td class="tr-role">{{ saveUserData.profile.role }}</td>
                    <td class="tr-pa">{{ saveUserData.profile.created }}</td>
                    <td class="tr-pa">{{ saveUserData.profile.updated }}</td>
                    <td class="tr-btns ">
                        <button class="settings pa__settings" @click="showEditWindow(saveUserData.id)"></button>
                    </td>
                </tr>
                <TheEditUser v-if="editUserVis" @closeEditWindow="closeEditWindow" :userIdDel="userIdDel"
                    @editUserFromUserList="editUserFromUserList" :roleChange="roleChange" />
            </tbody>
        </table>
    </div>
</template>

<script>

import TheEditUser from './TheUserListComponents/TheEditUser.vue'

export default {
    components: {
        TheEditUser
    },
    data() {
        return {
            editUserVis: false,
            userIdDel: null,
            roleChange: false
        }
    },
    props: {
        saveUserData: Object
    },
    methods: {
        showEditWindow(id) {
            this.editUserVis = true;
            this.userIdDel = id;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        closeEditWindow(data) {
            this.editUserVis = data; // закрытие окна «Обновление пользователя»
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        editUserFromUserList(data) { // обновить информацию о юзере, который отредактировали 
            this.$emit('editAuthorizedUser', data);
        }
    }
}
</script>

<style>
.pa__titles th {
    color: rgba(14, 22, 38, 0.5);
    padding: 0;
}

.pa__line td {
    padding: 0px;
}

.pa__settings {
    width: 26px !important;
    height: 26px !important;
    background-size: contain;
}

.pa__line:hover {
    background-color: transparent;
}
</style>