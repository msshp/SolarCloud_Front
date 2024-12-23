<template>
    <div class="window-container window-delete">
        <div class="window" v-if="editWindow">
            <h3>Редактирование пользователя</h3>
            <div class="dropdown">
                <button v-if="roleChange" class="dropdown__button" @click="chooseAccessLevel()"
                    v-bind:class="{ opened_button: selectorVisible }">{{ selectorContent }}<i class="white_delta"
                        v-bind:class="{ inverted_white_delta: selectorVisible }"></i></button>
                <ul v-if="roleChange" class="dropdown__list" v-bind:class="{ dropdown__list_visible: selectorVisible }">
                    <div></div>
                    <li class="dropdown__list-item" @click="chooseNewObserver()">Наблюдатель</li>
                    <li class="dropdown__list-item" @click="chooseNewModer()">Модератор</li>
                    <li class="dropdown__list-item" @click="chooseNewAdmin()">Администратор</li>
                </ul>

                <p>Имя</p>
                <input type="text" v-model.trim="newFirstName" placeholder="Имя">
                <p>Фамилия</p>
                <input type="text" v-model.trim="newLastName" placeholder="Фамилия">
                <p>Email</p>
                <input type="email" v-model.trim="newUserEmail" placeholder="Email">
                <p>Логин</p>
                <input type="text" v-model.trim="newUserLogin" placeholder="Логин">
                <!-- <input type="password" v-model.trim="newUserPass" placeholder="Пароль"> -->
                <div class="btn-container">
                    <button className="save-btn edit-save-btn" @click="sendNewUser()">Сохранить</button>
                    <button className="cancel-btn" @click="closeEditWindow">Отменить</button>
                </div>
            </div>
        </div>
        <div class="window" v-if="userEditError">
            <h3>{{ addErrorText }}</h3>
            <div class="btn-container">
                <button className="cancel-btn" @click="closeEditWindow">Выход</button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            selectorContent: 'Уровень доступа',
            engSelectorContent: '',
            selectorVisible: false,

            // v-if
            editWindow: true,
            userEditError: false,

            addErrorText: 'Ошибка',

            // поля
            newFirstName: '',
            newLastName: '',
            newUserEmail: '',
            newUserLogin: '',

            editableUser: {
                // "id": 1,
                // "email": "example@mail.com",
                // "username": "user_name",
                // "first_name": "Имя",
                // "last_name": "Фамилия",
                // "profile": {
                //     "id": 1,
                //     "role": "User",
                //     "telegram_chat_id": null,
                //     "created": "17.01.2024,09:42:54",
                //     "updated": "17.01.2024,09:42:54",
                //     "account": 20
                // }
            },
            newInfoUser: {}
        }
    },
    props: {
        userIdDel: Text,
        roleChange: Boolean
    },
    methods: {
        chooseAccessLevel() {
            this.selectorVisible = !this.selectorVisible;
        },
        chooseNewObserver() {
            this.selectorContent = 'Наблюдатель';
            this.engSelectorContent = 'User';
            this.selectorVisible = false;
        },
        chooseNewModer() {
            this.selectorContent = 'Модератор';
            this.engSelectorContent = 'Moderator';
            this.selectorVisible = false;
        },
        chooseNewAdmin() {
            this.selectorContent = 'Администратор';
            this.engSelectorContent = 'Admin';
            this.selectorVisible = false;
        },
        sendNewUser() {
            if (this.roleChange) {
                this.newInfoUser = {
                    "email": this.newUserEmail,
                    "username": this.newUserLogin,
                    "first_name": this.newFirstName,
                    "last_name": this.newLastName,
                    "profile": {
                        "role": this.engSelectorContent,
                        "account": this.editableUser.profile.account,
                        "telegram_chat_id": null
                    }
                }
            } else {
                this.newInfoUser = {
                    "email": this.newUserEmail,
                    "username": this.newUserLogin,
                    "first_name": this.newFirstName,
                    "last_name": this.newLastName,
                    "profile": {
                        "account": this.editableUser.profile.account,
                    }
                }
            }

            // отправка новых данных
            axios.patch(`http://cloud.io-tech.ru/api/users/${this.userIdDel}/`, this.newInfoUser, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 200) { // данные обновлены

                    this.$emit('editUserFromUserList', response.data); // изменить строку в таблице с пользователями

                    // сообщение об успешном выводе
                    this.addErrorText = 'Данные успешно обновлены';
                    this.errorWindow();

                } else if (response.status === 401) { // окно с ошибкой
                    this.errorWindow();
                } else if (response.status === 403) {
                    this.errorWindow();
                } else if (response.status === 404) {
                    this.errorWindow();
                } else {
                    this.errorWindow();
                }
            }).catch((error) => {
                console.log(error);
                this.addErrorText = error.request.responseText;
                this.errorWindow();
            });
        },
        errorWindow() {
            this.userEditError = true;
            this.editWindow = false;
        },
        closeEditWindow() {
            this.$emit('closeEditWindow', false);
        }
    },
    mounted() {
        // запрос данных пользователя
        axios.get(`http://cloud.io-tech.ru/api/users/${this.userIdDel}/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {

                this.editableUser = response.data; // сохранение данных юзера

                // заполнение полей

                if (this.editableUser.profile.role === 'User') {
                    this.chooseNewObserver();
                } else if (this.editableUser.profile.role === 'Moderator') {
                    this.chooseNewModer();
                } else if (this.editableUser.profile.role === 'Admin') {
                    this.chooseNewAdmin();
                }

                this.newFirstName = this.editableUser.first_name;
                this.newLastName = this.editableUser.last_name;
                this.newUserEmail = this.editableUser.email;
                this.newUserLogin = this.editableUser.username;
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    }
}
</script>

<style>
.window {
    width: 590px;
    background-color: #F8F6F4;
    border-radius: 8px;
    padding: 48px;
}

.result_window {
    width: 320px;
    margin: 0 auto 24px;
    font-weight: 400;
    font-size: 16px;
    line-height: 112%;
    color: #0e1626;
}

h3 {
    font-weight: 500;
    font-size: 20px;
    line-height: 100%;
    letter-spacing: -0.02em;
    color: #0e1626;
    text-align: center;
    margin: 0px 0px 32px;
}

.dropdown {
    position: relative;
    display: flex;
    flex-direction: column;
}

.dropdown__button {
    height: 48px;
    width: 100%;
    text-align: left;
    border-radius: 8px;
    background: #294b8e;
    padding: 8px 20px 8px 16px;
    margin-top: 8px;
    margin-bottom: 8px;
    font-weight: 500;
    font-size: 16px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #f8f6f4;

    display: flex;
    justify-content: space-between;
    align-items: center;
}

.dropdown__button i {
    width: 20px;
    height: 20px;
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
}

.dropdown__list {
    display: none;
    position: absolute;
    left: 0;
    top: 48px;
    list-style-type: none;
    background: #294b8e;
    border-radius: 0 0 8px 8px;
    width: 100%;
    padding: 0 0 4px 0;
    margin: 0;
}

ul div {
    border: 0.3px solid #f8f6f4;
    margin: 8px 20px 0;
    margin-bottom: 12px;
}

.dropdown__list-item {
    text-align: left;
    border-radius: 8px;
    height: 40px;
    cursor: pointer;
    font-weight: 400;
    font-size: 16px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #f8f6f4;
    margin: 0 20px 8px;
    display: flex;
    align-items: center;
    padding-left: 10px;
}

.dropdown__list-item:hover {
    color: #294b8e;
    background-color: #f8f6f4;
}

.dropdown input {
    font-weight: 500;
    width: 94%;
}

.dropdown input::placeholder {
    color: #6f7e9e;
}

.btn-container {
    display: flex;
    justify-content: center;
    margin-top: 8px;
}

.edit-save-btn {
    width: 200px !important;
}

.cancel-btn {
    background-color: transparent;
    color: #294b8e;
    border: 1px solid #294b8e;
}

.dropdown p {
    margin: 10px 17px;
    color: #293b5f;
    font-weight: 500;
}
</style>