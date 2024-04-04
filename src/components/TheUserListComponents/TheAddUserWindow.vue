<template>
    <div class="window-container">
        <div class="window" v-if="addWindow">
            <h3>Добавление пользователя</h3>
            <div class="dropdown">
                <button class="dropdown__button" @click="chooseAccessLevel()"
                    v-bind:class="{ opened_button: selectorVisible }">{{ selectorContent }}<i class="white_delta"
                        v-bind:class="{ inverted_white_delta: selectorVisible }"></i></button>
                <ul class="dropdown__list" v-bind:class="{ dropdown__list_visible: selectorVisible }">
                    <div></div>
                    <li class="dropdown__list-item" @click="chooseNewObserver()">Наблюдатель</li>
                    <li class="dropdown__list-item" @click="chooseNewModer()">Модератор</li>
                    <li class="dropdown__list-item" @click="chooseNewAdmin()">Администратор</li>
                </ul>
                <input type="text" v-model.trim="newFirstName" placeholder="Имя">
                <input type="text" v-model.trim="newLastName" placeholder="Фамилия">
                <input type="email" v-model.trim="newUserEmail" placeholder="Email">
                <input type="text" v-model.trim="newUserLogin" placeholder="Логин">
                <input type="password" v-model.trim="newUserPass" placeholder="Пароль">
                <div class="btn-container">
                    <button className="save-btn" @click="sendNewUser()">Сохранить</button>
                    <button className="cancel-btn" @click="closeAddWindow">Отменить</button>
                </div>
            </div>
        </div>
        <div class="window" v-if="userAddedSuccessfully">
            <h3>Пользователь добавлен</h3>
            <div class="result_window">
                <p>Email: {{ resultNewUserEmail }}</p>
                <p>Пароль: {{ resultNewUserPass }}</p>
            </div>
            <div class="btn-container">
                <button className="save-btn" @click="openAddWindow()">Добавить ещё</button>
                <button className="cancel-btn" @click="closeAddWindow">Выход</button>
            </div>
        </div>
        <div class="window" v-if="userAddedError">
            <h3>{{ addErrorText }}</h3>
            <div class="btn-container">
                <button className="cancel-btn" @click="closeAddWindow">Выход</button>
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
            engSelectorContent: 'User',
            selectorVisible: false,

            // v-if
            userAddedSuccessfully: false,
            addWindow: true,
            userAddedError: false,

            // result

            resultNewUserEmail: '',
            resultNewUserPass: '',

            addErrorText: 'Ошибка',

            newUserPass: '',
            newUserLogin: '',
            newUserEmail: '',
            newFirstName: '',
            newLastName: ''
        }
    },
    props: {
        saveUserData: Object
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
            axios.post('http://cloud.io-tech.ru/api/users/',
                {
                    "email": this.newUserEmail,
                    "username": this.newUserLogin,
                    "first_name": this.newFirstName,
                    "last_name": this.newLastName,
                    "password": this.newUserPass,
                    "profile": {
                        "role": this.engSelectorContent,
                        "account": this.saveUserData.profile.account,
                        "telegram_chat_id": null
                    }
                }, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 201) { // пользователь успешно создан

                    this.$emit('newUserLine', response.data)

                    this.resultNewUserEmail = this.newUserEmail; // вывод email и пароль в итоговом окне
                    this.resultNewUserPass = this.newUserPass;

                    this.newUserEmail = '';
                    this.newUserLogin = '';
                    this.newFirstName = '';
                    this.newLastName = '';
                    this.newUserPass = '';
                    this.engSelectorContent = 'User';
                    this.selectorContent = 'Уровень доступа';

                    this.selectorVisible = false;

                    // сообщение об успешном выводе

                    this.userAddedSuccessfully = true;
                    this.addWindow = false;

                } else if (response.status === 400) { // окно с ошибкой

                    this.errorWindow();
                } else if (response.status === 401) {
                    this.errorWindow();

                } else if (response.status === 403) {

                    this.errorWindow();
                } else {

                    this.errorWindow();
                }
            }).catch((error) => {
                this.addErrorText = error.request.responseText;
                this.errorWindow();
            });
        },
        errorWindow() {
            this.userAddedError = true;
            this.addWindow = false;
        },
        closeAddWindow() {
            this.$emit('closeAddWindow', false);
            // очистить поля все
            this.newUserEmail = '';
            this.newUserLogin = '';
            this.newFirstName = '';
            this.newLastName = '';
            this.newUserPass = '';
            this.engSelectorContent = 'User';
            this.selectorContent = 'Уровень доступа';

            this.selectorVisible = false;
            this.userAddedError = false;

            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openAddWindow() {
            this.userAddedSuccessfully = false;
            this.addWindow = true;
        }
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
    padding: 14px 16px;
    border-radius: 8px;
    background-color: transparent;
    border: 1px solid #293b5f;
    color: #293b5f;
    font-size: 16px;
    margin-bottom: 8px;
}

.dropdown input::placeholder {
    color: #6f7e9e;
}

.btn-container {
    display: flex;
    justify-content: center;
    margin-top: 8px;
}

.btn-container button {
    border-radius: 8px;
    width: 156px;
    height: 48px;
    font-weight: 500;
    font-size: 16px;
    line-height: 112%;
    text-align: center;
}

.save-btn {
    background: #294b8e;
    color: #f8f6f4;
    margin-right: 8px;
}

.cancel-btn {
    background-color: transparent;
    color: #294b8e;
    border: 1px solid #294b8e;
}
</style>