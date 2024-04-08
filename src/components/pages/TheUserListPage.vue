<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Список пользователей</p>
            <div></div>
        </div>
        <div v-if="access" class="account__number-users">
            <div class="role-block">
                <div class="role-block__admin"></div>
                <div>
                    <p class="role-name">администратор</p>
                    <p class="role-count"> {{ numAdmin }}</p>
                </div>
            </div>
            <div class="role-block">
                <div class="role-block__moder"></div>
                <div>
                    <p class="role-name">модератор</p>
                    <p class="role-count">{{ numModer }}</p>
                </div>
            </div>
            <div class="role-block">
                <div class="role-block__moder"></div>
                <div>
                    <p class="role-name">наблюдатель</p>
                    <p class="role-count">{{ numObserver }}</p>
                </div>
            </div>
        </div>
        <button v-if="access" class="account__add-users" @click="addUser()"><span>+</span> Добавить
            пользователя</button>
        <div v-if="access" class="account-separator"></div>
        <table>
            <thead>
                <tr>
                    <th class="tr-id"><input v-model.trim="searchUserID" type="number" placeholder="ID"
                            v-on:input="searchUser">
                    </th>
                    <th><input v-model.trim="searchUserName" type="text" placeholder="Имя"
                            v-on:input="searchUserByName"></th>
                    <th><input v-model.trim="searchUserEmail" type="text" placeholder="email"
                            v-on:input="searchUserByEmail"></th>
                    <th class="select_role">
                        <button class="select__button" @click="selectRole()"
                            v-bind:class="{ opened_button: selectorTableVisible }">{{ selectorTableContent }}<i
                                class="delta" v-bind:class="{ inverted_delta: selectorTableVisible }"></i></button>
                        <ul class="dropdown__list_table"
                            v-bind:class="{ dropdown__list_visible: selectorTableVisible }">
                            <li class="dropdown__list-item_table" @click="selectTableAllRoles()">все роли</li>
                            <li class="dropdown__list-item_table" @click="selectTableObserver()">наблюдатель</li>
                            <li class="dropdown__list-item_table" @click="selectTableModer()">модератор</li>
                            <li class="dropdown__list-item_table" @click="selectTableAdmin()">администратор</li>
                        </ul>
                    </th>
                    <th class="sort-by-date">дата создания</th>
                    <th class="sort-by-date">дата обновления</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                <!-- первая строка всегда авторизованный user -->
                <tr>
                    <td class="tr-id">{{ saveUserData.id }}</td>
                    <td class="tr-name"> {{ saveUserData.first_name + ' ' + saveUserData.last_name }}</td>
                    <td>{{ saveUserData.email }}</td>
                    <td>{{ saveUserData.profile.role }}</td>
                    <td>{{ saveUserData.profile.created }}</td>
                    <td>{{ saveUserData.profile.updated }}</td>
                    <td class="tr-btns"></td>
                </tr>
                <TheAccountLine :usersList="usersList" @deleteUserFromMainUserList="deleteUserFromMainUserList"
                    @editUserFromMainUserList="editUserFromMainUserList" />
            </tbody>
        </table>
    </div>
    <TheAddUserWindow v-if="addUserWindowVis" @closeAddWindow="closeAddWindow" :saveUserData="saveUserData"
        @newUserLine="addNewUserToList" />
</template>

<script>

// import { containsProp } from '@vueuse/core';
import TheAccountLine from '../TheUserListComponents/TheAccountLine.vue'
import TheAddUserWindow from '../TheUserListComponents/TheAddUserWindow.vue'

import axios from 'axios';

export default {
    components: {
        TheAccountLine,
        TheAddUserWindow
    },
    props: {
        saveUserData: Object,
        access: Boolean,
    },
    data() {
        return {
            addUserWindowVis: false,

            usersList: [],
            numAdmin: 0,
            numModer: 0,
            numObserver: 0,

            selectorTableContent: 'роль',
            selectorTableVisible: false,

            choosingSortByCreation: false,

            searchUserEmail: '',
            searchUserName: '',
            searchUserID: ''
        }
    },
    methods: {
        deleteUserFromMainUserList(id) {
            this.usersList = this.usersList.filter((user) => user.id !== id);
            this.userRecalculation();
        },
        editUserFromMainUserList() {
            this.getUsersList();
        },
        addUser() {
            this.addUserWindowVis = true;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        userRecalculation() {
            this.numModer = 0;
            this.numObserver = 0;
            this.numAdmin = 0;

            this.usersList.forEach((el) => {
                if (el.profile.role === 'User' || el.profile.role === 'наблюдатель') {
                    el.profile.role = 'наблюдатель';
                    this.numObserver++;
                } else if (el.profile.role === 'Admin' || el.profile.role === 'администратор') {
                    el.profile.role = 'администратор';
                    this.numAdmin++;
                } else if (el.profile.role === 'Moderator' || el.profile.role === 'модератор') {
                    el.profile.role = 'модератор';
                    this.numModer++;
                }
            })

            this.numAdmin++; // первая строка таблицы
        },
        closeAddWindow(data) {
            this.addUserWindowVis = data; // закрытие окна «Добавление пользователя»
        },
        addNewUserToList() {
            this.getUsersList();
        },
        searchUser() { // поиск пользователя по ID
            let list = document.querySelectorAll('tbody tr');

            if (this.searchUserID != '') {
                list.forEach(elem => {
                    if (elem.firstChild.innerText.search(this.searchUserID) === -1) {
                        elem.classList.add('display_none');
                    } else {
                        elem.classList.remove('display_none');
                    }
                });
            } else {
                list.forEach(elem => {
                    elem.classList.remove('display_none');
                });
            }
        },
        searchUserByName() {
            let list = document.querySelectorAll('tbody tr');

            if (this.searchUserName != '') {
                list.forEach(elem => {
                    if (elem.children[1].innerText.search(this.searchUserName) === -1) {
                        elem.classList.add('display_none');
                    } else {
                        elem.classList.remove('display_none');
                    }
                });
            } else {
                list.forEach(elem => {
                    elem.classList.remove('display_none');
                });
            }
        },
        searchUserByEmail() {
            let list = document.querySelectorAll('tbody tr');

            if (this.searchUserEmail != '') {
                list.forEach(elem => {
                    if (elem.children[2].innerText.search(this.searchUserEmail) === -1) {
                        elem.classList.add('display_none');
                    } else {
                        elem.classList.remove('display_none');
                    }
                });
            } else {
                list.forEach(elem => {
                    elem.classList.remove('display_none');
                });
            }
        },
        selectRole() {
            this.selectorTableVisible = !this.selectorTableVisible;
        },
        selectTableObserver() {
            this.selectorTableContent = 'наблюдатель';
            this.universalSortByRole();
        },
        universalSortByRole() {
            this.selectorTableVisible = false;
            let list = document.querySelectorAll('tbody tr');

            list.forEach(elem => {
                if (elem.children[3].innerText.search(this.selectorTableContent) === -1) {
                    elem.classList.add('display_none');
                } else {
                    elem.classList.remove('display_none');
                }
            });
        },
        selectTableModer() {
            this.selectorTableContent = 'модератор';
            this.universalSortByRole();
        },
        selectTableAdmin() {
            this.selectorTableContent = 'администратор';
            this.universalSortByRole();
        },
        selectTableAllRoles() {
            this.selectorTableContent = 'все роли';
            this.selectorTableVisible = false;

            let list = document.querySelectorAll('tbody tr');
            list.forEach(elem => {
                elem.classList.remove('display_none');
            });
        },
        getUsersList() {
            axios.get('http://cloud.io-tech.ru/api/users/?limit=100',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.usersList = response.data.results;

                    this.usersList.sort(function (a, b) { // сортировка usersList по id
                        if (a.id > b.id) {
                            return 1;
                        }
                        if (a.id < b.id) {
                            return -1;
                        }
                        return 0;
                    });

                    // удаление запятой в датах
                    this.usersList.forEach((el) => {
                        el.profile.created = el.profile.created.replace(',', ' ');
                        el.profile.updated = el.profile.updated.replace(',', ' ');
                    })

                    if (this.saveUserData.profile.role === 'администратор' || this.saveUserData.profile.role === 'Admin') {
                        this.usersList.shift(); // удаление первого пользователя в списке (дублируется строка) - только у админа
                    }

                    this.userRecalculation();

                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        }
    },
    // вывести список пользователей аккаунта
    mounted() {
        this.getUsersList();
    }
}
</script>

<style>
.account-separator {
    margin-bottom: 16px !important;
}

.account__number-users {
    display: flex;
}

.role-block {
    border-radius: 8px;
    width: 310px;
    height: 140px;
    background: #f8f6f4;
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-right: 24px;
    margin-bottom: 24px;
}

.role-block div:first-child {
    margin: 16px 0 16px 16px;
    width: 105px;
    height: 105px;
    background-repeat: no-repeat;
    border-radius: 8px;
}

.role-block div:last-child {
    margin-right: 32px;
    text-align: right;
}

.role-block__admin {
    background-image: url(../img/account/admin.jpg);
}

.role-block__moder {
    background-image: url(../img/account/moder.jpg);
}

.role-name {
    font-weight: 400;
    font-size: 16px;
    line-height: 112%;
    color: #293b5f;
}

.role-count {
    font-weight: 500;
    font-size: 25px;
    line-height: 42%;
    letter-spacing: -0.01em;
    color: #0e1626;
}

.account__add-users {
    border-radius: 8px;
    width: 310px;
    height: 47px;
    background: #f4ca8d;
    font-weight: 500;
    font-size: 16px;
    line-height: 112%;
    color: #0e1626;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 24px;
}

.account__add-users span {
    font-weight: 400;
    font-size: 32px;
    line-height: 56%;
    color: #0e1626;
    margin-right: 12px;
}

table {
    width: 100%;
}

table input,
.select__button,
.sort-by-creation {
    width: 85%;
    background-color: transparent;
    color: #0E1626;
    border: 1px solid rgba(14, 22, 38, 0.5);
    padding: 6px 8px;
    border-radius: 8px;
    font-weight: 600;
    font-size: 16px;
    line-height: 112%;
}

.sort-by-creation {
    color: #7e828a;
}

.select__button i {
    display: block;
    width: 16px;
}

.select__button {
    color: rgba(14, 22, 38, 0.5);
    width: 100%;
    text-align: left;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* 
table select {
    color: #919eb8;
} */

table input::placeholder,
table select {
    color: rgba(14, 22, 38, 0.5);
}

thead th:last-child {
    width: 90px;
}

tbody td {
    padding: 0 0 0 16px;
    font-size: 14px;
}

td p {
    margin: 0;
}

.tr-btns {
    padding-left: 4px;
}

.tr-btns button {
    width: 32px;
    height: 32px;
    background-repeat: no-repeat;
    background-color: transparent;
}

.select_role {
    position: relative;
    display: flex;
    flex-direction: column;
    padding-left: 8px !important;
    margin-bottom: 0px !important;
}

.dropdown__list_table {
    display: none;
    position: absolute;
    left: 8px;
    top: 32px;
    list-style-type: none;
    background: #294b8e;
    border-radius: 2px 2px 8px 8px;
    width: 94%;
    padding: 4px 0;
    margin: 0;
}

.dropdown__list-item_table {
    text-align: left;
    border-radius: 8px;
    height: 36px;
    cursor: pointer;
    font-weight: 400;
    font-size: 14px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #f8f6f4;
    display: flex;
    align-items: center;
    padding-left: 10px;
}

.dropdown__list-item_table:hover {
    color: #294b8e;
    background-color: #f8f6f4;
}

.grey_btn {
    background-color: #7e828a;
    color: #f8f6f4;
}

.sort-by-date {
    color: rgba(14, 22, 38, 0.5);
    text-align: left;
    padding-left: 16px !important;
}

tr th input {
    margin-left: 12px;
}

@media (min-width: 1775px) {
    .dropdown__list_table {
        width: 96%;
    }
}
</style>