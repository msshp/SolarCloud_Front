<template>
    <div v-if="loginPageVisibility" class="content"
        style="width: 100%;height: 100%; vertical-align: center; position: fixed;">
        <div v-if="loginFormVisibility" class="form">
            <div className="logo-container"><img src="../public/img/logo/logo_solar_white_rus_crop.svg" alt="logo">
            </div>
            <input type="email" v-model.trim="userEmail" v-bind:class="{ empty_input: userEmailIsEmpty }"
                placeholder="Email" autocomplete="current-password" required>
            <input class="last-input" type="password" v-model.trim="userPass"
                v-bind:class="{ empty_input: userPassIsEmpty }" placeholder="Пароль" autocomplete="current-password"
                required>
            <button className="login-btn" @click="logIn()">Войти</button>
            <div class="form-btns__container"><button className="recover-password"
                    @click="RecoverPassword()">Восстановить пароль</button><button className="reg-btn"
                    @click="logRegToggle()">Создать
                    проект</button>
            </div>
        </div>
        <div v-if="errorLoginVisibility" class="form">
            <p class="reg-title">{{ errorLoginMessage }}</p>
            <button v-if="errorBtnVisibility" className="login-btn" @click="tryLogIn()">Попробовать снова</button>
        </div>
        <div v-if="regFormVisibility" class="form">
            <input type="text" v-model="projName" v-bind:class="{ empty_input: projNameIsEmpty }"
                placeholder="Название проекта">
            <input type="text" v-model="projDescription" v-bind:class="{ empty_input: projDescriptionIsEmpty }"
                placeholder="Описание проекта">
            <input type="text" v-model.trim="projUserName" v-bind:class="{ empty_input: projUserNameIsEmpty }"
                placeholder="Имя пользователя">
            <input type="text" v-model.trim="projUserSurname" v-bind:class="{ empty_input: projUserSurnameIsEmpty }"
                placeholder="Фамилия пользователя">
            <input type="email" v-model.trim="projUserEmail" v-bind:class="{ empty_input: projUserEmailIsEmpty }"
                placeholder="Email" autocomplete="username">
            <input type="text" v-model.trim="projUserLogin" v-bind:class="{ empty_input: projUserLoginIsEmpty }"
                placeholder="Логин" required>
            <input class="last-input" type="password" v-model.trim="projUserPass"
                v-bind:class="{ empty_input: projUserPassIsEmpty }" placeholder="Пароль" autocomplete="new-password"
                required>
            <button className="login-btn" @click="createProj()">Создать проект</button>
            <button className="reg-btn reg-btn_back" @click="logRegToggle()">Назад</button>
        </div>
        <div v-if="resetPasswordVisibility" class="form">
            <p class="reset-p">Введите электронный адрес</p>
            <input type="email" v-model.trim="resetUserEmail" v-bind:class="{ empty_input: resetUserEmailIsEmpty }"
                placeholder="Email">
            <button className="login-btn" @click="sendEmail()">Отправить</button>
            <button className="reg-btn reg-btn_back" @click="resetBack()">Назад</button>
        </div>
        <div v-if="regMessageVisibility" class="form">
            <p class="reg-title">Вы зарегистрированы</p>
            <button className="login-btn" @click="closeRegMessage()">Войти</button>
        </div>
        <div v-if="errorMessageVisibility" class="form">
            <p class="reg-title">Ошибка</p>
            <button className="login-btn" @click="tryRegWindow()">Попробовать снова</button>
        </div>
        <div v-if="errorResetVisibility" class="form">
            <p class="reg-title">Ошибка</p>
            <button className="login-btn" @click="RecoverPassword()">Попробовать снова</button>
        </div>
    </div>

    <div v-if="mainPageVisibility">
        <div className="wrapper">
            <div class="sidenav" v-bind:class="{ sidenav_hidden: sidenavIsHidden }">
                <div className="side-menu__btn" @click="sidenavToggle()">
                    <div className="icon-burger"></div>
                </div>
                <div v-if="pages.controllerPageVisibility" @click="openList()" className="side-menu__btn-back"></div>
                <div className="sidenav__top-block">
                    <img src="../public/img/logo/logo_solar_blue_rus_crop.svg" alt="logo">
                </div>
                <nav className="sidenav__block">
                    <button id="sidenav__lk" @click="openPersonalArea()"
                        v-bind:class="{ sidebtn_active: pages.personalAreaPageVisibility }">
                        <div class="icon-lk" v-bind:class="{ iconlk_active: pages.personalAreaPageVisibility }">
                        </div>
                    </button>
                    <button id="sidenav__control" @click="openControl()"
                        v-bind:class="{ sidebtn_active: pages.controlPageVisibility }">
                        <div class="icon-control" v-bind:class="{ iconcontrol_active: pages.controlPageVisibility }">
                        </div><span>Контрольная панель</span>
                    </button>
                    <div v-if="projBtnVisibility" class="separator"></div>
                    <button v-if="projBtnVisibility" id="sidenav__project" @click="openProject()"
                        v-bind:class="{ sidebtn_active: pages.projectPageVisibility }">
                        <div class="icon-project" v-bind:class="{ iconproj_active: pages.projectPageVisibility }"></div>
                        <span>Проект</span>
                    </button>
                    <button v-if="listUsersBtnVisibility" id="sidenav__userlist" @click="openUserListPage()"
                        v-bind:class="{ sidebtn_active: pages.listUsersPageVisibility }">
                        <div class="icon-personal-area"
                            v-bind:class="{ icon_personal_area_active: pages.listUsersPageVisibility }">
                        </div><span>Список пользователей</span>
                    </button>
                    <div class="separator"></div>
                    <button id="sidenav__map" @click="openMap()"
                        v-bind:class="{ sidebtn_active: pages.mapPageVisibility }">
                        <div class="icon-map" v-bind:class="{ iconmap_active: pages.mapPageVisibility }"></div>
                        <span>Карта</span>
                    </button>
                    <button id="sidenav__list" @click="openList()"
                        v-bind:class="{ sidebtn_active: pages.listPageVisibility }">
                        <div class="icon-list" v-bind:class="{ iconlist_active: pages.listPageVisibility }"></div>
                        <span>Список
                            контроллеров</span>
                    </button>
                    <!-- <button id="sidenav__subscription" @click="openSubscription()"
                        v-bind:class="{ sidebtn_active: pages.subscriptionPageVisibility }">
                        <div class="icon-subscription"
                            v-bind:class="{ iconsub_active: pages.subscriptionPageVisibility }"></div><span>Подписка</span>
                    </button> -->
                    <div class="separator"></div>
                    <button id="sidenav__events" @click="openEvents()"
                        v-bind:class="{ sidebtn_active: pages.eventsPageVisibility }">
                        <div class="icon-events" v-bind:class="{ iconevents_active: pages.eventsPageVisibility }"></div>
                        <span>События</span>
                    </button>
                    <button id="sidenav__sidelogout" @click="logOut()">
                        <div class="icon-logout"></div><span>Выход</span>
                    </button>
                    <!-- <button id="sidenav__commands" @click="openCommands()"
                        v-bind:class="{ sidebtn_active: pages.commandsPageVisibility }">
                        <div class="icon-commands" v-bind:class="{ iconcom_active: pages.commandsPageVisibility }">
                        </div><span>Команды</span>
                    </button> -->
                    <!-- <button id="sidenav__reports" @click="openReports()"
                        v-bind:class="{ sidebtn_active: pages.reportsPageVisibility }">
                        <div class="icon-reports" v-bind:class="{ iconreports_active: pages.reportsPageVisibility }">
                        </div><span>Отчёты</span>
                    </button> -->
                </nav>
            </div>
            <div id="page-content" class="page-content" v-bind:class="{ content_compressed: contentIsCompressed }">
                <nav class="top-menu" v-bind:class="{ top_menu_compressed: contentIsCompressed }">
                    <div class="top-menu__item" @click="openExtraMenu()"
                        v-bind:class="{ personalarea_active: pages.personalAreaPageVisibility }">
                        <div class="top-menu__item-name">
                            <p class="menu__user-name"
                                v-bind:class="{ personalarea_active: pages.personalAreaPageVisibility }"> {{
                                    saveUserData.first_name + ' ' + saveUserData.last_name }}</p>
                            <p class="menu__user-role"
                                v-bind:class="{ personalarea_active: pages.personalAreaPageVisibility }">
                                {{ saveUserData.profile.role }}
                            </p>
                        </div>
                        <div class="delta-block">
                            <div class="delta"
                                v-bind:class="{ inverted_delta: extraMenuIsHidden, white_delta: pages.personalAreaPageVisibility, inverted_white_delta: extraMenuIsHidden && pages.personalAreaPageVisibility }">
                            </div>
                        </div>
                        <nav class="top-menu__item-options" v-bind:class="{ display_flex: extraMenuIsHidden }">
                            <button id="button-pa" @click="openPersonalArea()"><i
                                    className="icon-personal-area"></i>Личный кабинет</button>
                            <button @click="logOut()" id="button-logout"><i className="icon-logout"></i>Выйти из
                                аккаунта</button>
                        </nav>
                    </div>
                </nav>
                <TheControlPage v-if="pages.controlPageVisibility" />
                <TheProjectPage v-if="pages.projectPageVisibility" />
                <TheUserListPage v-if="pages.listUsersPageVisibility" :saveUserData="saveUserData" :access="access" />
                <TheMapPage v-if="pages.mapPageVisibility" :saveUserData="saveUserData" :controllerList="controllerList"
                    @openMainControllerPage="openMainControllerPage" />
                <TheListPage v-if="pages.listPageVisibility" @openMainControllerPage="openMainControllerPage"
                    :access="access" :saveUserData="saveUserData" :controllerList="controllerList"
                    @deleteControllerFromMainList="deleteControllerFromMainList"
                    @addControllerToMainList="addControllerToMainList" />
                <ThePersonalArea v-if="pages.personalAreaPageVisibility" :saveUserData="saveUserData"
                    @editAuthorizedUser="editAuthorizedUser" />
                <TheSubscriptionPage v-if="pages.subscriptionPageVisibility" />
                <TheEventsPage v-if="pages.eventsPageVisibility" />
                <TheCommandsPage v-if="pages.commandsPageVisibility" />
                <TheReportsPage v-if="pages.reportsPageVisibility" />
                <TheControllerPage v-if="pages.controllerPageVisibility" :controllerId="controllerId" :access="access"
                    @deleteControllerFromList="deleteControllerFromList" />
            </div>
        </div>
    </div>
</template>

<script>

import TheMapPage from './components/TheMapPage.vue'
import ThePersonalArea from './components/ThePersonalArea.vue'
import TheProjectPage from './components/TheProjectPage.vue'
import TheUserListPage from './components/pages/TheUserListPage.vue'
import TheControlPage from './components/TheControlPage.vue'
import TheListPage from './components/TheListPage.vue'
import TheSubscriptionPage from './components/TheSubscriptionPage.vue'
import TheEventsPage from './components/TheEventsPage.vue'
import TheCommandsPage from './components/TheCommandsPage.vue'
import TheReportsPage from './components/TheReportsPage.vue'
import TheControllerPage from './components/TheControllerPage.vue'

import axios from 'axios';

export default {
    components: {
        TheMapPage,
        ThePersonalArea,
        TheProjectPage,
        TheUserListPage,
        TheControlPage,
        TheListPage,
        TheSubscriptionPage,
        TheEventsPage,
        TheCommandsPage,
        TheReportsPage,
        TheControllerPage
    },
    data() {
        return {
            loginFormVisibility: true,
            regFormVisibility: false,
            resetPasswordVisibility: false,
            regMessageVisibility: false,
            errorMessageVisibility: false,
            errorResetVisibility: false,
            errorLoginVisibility: false,
            errorLoginMessage: 'Ошибка',
            errorBtnVisibility: true,

            mainPageVisibility: false, // доступ после авторизации
            loginPageVisibility: true, // страница авторизации

            userEmail: '',
            userPass: '',

            projName: '',
            projDescription: '',
            projUserName: '',
            projUserSurname: '',
            projUserEmail: '',
            projUserLogin: '',
            projUserPass: '',

            resetUserEmail: '',
            resetUserEmailIsEmpty: false,

            userEmailIsEmpty: false,
            userPassIsEmpty: false,
            projNameIsEmpty: false,
            projDescriptionIsEmpty: false,
            projUserNameIsEmpty: false,
            projUserSurnameIsEmpty: false,
            projUserEmailIsEmpty: false,
            projUserLoginIsEmpty: false,
            projUserPassIsEmpty: false,
            sidenavIsHidden: false,
            contentIsCompressed: false,
            extraMenuIsHidden: false,
            listUsersBtnVisibility: false,
            projBtnVisibility: false,

            pages: {
                personalAreaPageVisibility: false,
                mapPageVisibility: false,
                projectPageVisibility: false,
                controlPageVisibility: false,
                listPageVisibility: false,
                subscriptionPageVisibility: false,
                eventsPageVisibility: false,
                commandsPageVisibility: false,
                reportsPageVisibility: false,
                listUsersPageVisibility: false,
                controllerPageVisibility: false,
            },

            saveUserData: {
                "id": null,
                "email": "",
                "username": "",
                "first_name": "Имя",
                "last_name": "Фамилия",
                "profile": {
                    "id": null,
                    "role": "администратор",
                    "telegram_chat_id": null,
                    "created": "",
                    "updated": "",
                    "account": null
                }
            },
            access: true,
            controllerId: null,

            controllerList: [],
            logInIndication: false // индикатор для включения карты
        };
    },
    methods: {
        RecoverPassword() {
            this.resetPasswordVisibility = true;
            this.loginFormVisibility = false;
            this.errorResetVisibility = false;
        },
        resetBack() {
            this.resetPasswordVisibility = false;
            this.loginFormVisibility = true;
        },
        logRegToggle() {
            this.loginFormVisibility = !this.loginFormVisibility;
            this.regFormVisibility = !this.regFormVisibility;
        },
        createProj() {
            if (this.projName == undefined || this.projName.trim().length === 0) {
                this.projNameIsEmpty = true;
            }
            else {
                this.projNameIsEmpty = false;
            }
            if (this.projDescription == undefined || this.projDescription.trim().length === 0) {
                this.projDescriptionIsEmpty = true;
            }
            else {
                this.projDescriptionIsEmpty = false;
            }
            if (this.projUserName == undefined || this.projUserName.trim().length === 0) {
                this.projUserNameIsEmpty = true;
            }
            else {
                this.projUserNameIsEmpty = false;
            }
            if (this.projUserSurname == undefined || this.projUserSurname.trim().length === 0) {
                this.projUserSurnameIsEmpty = true;
            }
            else {
                this.projUserSurnameIsEmpty = false;
            }
            if (this.projUserEmail == undefined || this.projUserEmail.trim().length === 0) {
                this.projUserEmailIsEmpty = true;
            }
            else {
                this.projUserEmailIsEmpty = false;
            }
            if (this.projUserLogin == undefined || this.projUserLogin.trim().length === 0) {
                this.projUserLoginIsEmpty = true;
            }
            else {
                this.projUserLoginIsEmpty = false;
            }
            if (this.projUserPass == undefined || this.projUserPass.trim().length === 0) {
                this.projUserPassIsEmpty = true;
            }
            else {
                this.projUserPassIsEmpty = false;
            }
            if (!this.projNameIsEmpty && !this.projDescriptionIsEmpty && !this.projUserNameIsEmpty && !this.projUserSurnameIsEmpty && !this.projUserEmailIsEmpty && !this.projUserLoginIsEmpty && !this.projUserPassIsEmpty) {
                this.createProjectPost();
            }
        },
        createProjectPost() {
            axios.post('http://cloud.io-tech.ru/api/accounts/signup/',
                {
                    "name": this.projName,
                    "description": this.projDescription,
                    "account_type": 0,
                    "user": {
                        "email": this.projUserEmail,
                        "username": this.projUserLogin,
                        "first_name": this.projUserName,
                        "last_name": this.projUserSurname,
                        "password": this.projUserPass,
                        "profile": {
                            "role": "User"
                        }
                    }
                }, {
                headers: {
                    'Content-Type': 'application/json'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 201) {
                    this.regMessageVisibility = true;

                    this.projName = '';
                    this.projDescription = '';
                    this.projUserEmail = '';
                    this.projUserLogin = '';
                    this.projUserName = '';
                    this.projUserSurname = '';
                    this.projUserPass = '';
                } else if (response.status === 400) {
                    this.errorMessageVisibility = true;
                } else {
                    this.errorMessageVisibility = true;
                }
            }).catch((error) => {
                console.log(error);
                this.errorMessageVisibility = true;
            });

            this.regFormVisibility = false;

            if (this.regMessageVisibility === true) {
                // очистка полей в регистрации
                this.projName = '';
                this.projDescription = '';
                this.projUserEmail = '';
                this.projUserLogin = '';
                this.projUserName = '';
                this.projUserSurname = '';
                this.projUserPass = '';
            }
        },
        sendEmail() {
            axios.post('http://cloud.io-tech.ru/api/users/reset_password/',
                {
                    "email": this.resetUserEmail
                }, {
                headers: {
                    'Content-Type': 'application/json'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 204) {
                    this.resetPasswordVisibility = false;
                    this.errorBtnVisibility = false;

                    this.errorLoginMessage = `На ваш электронный адрес пришла ссылка. Перейдите по ней для ввода нового пароля.`;
                    this.drawErrorLogIn();
                } else if (response.status === 400) {
                    console.log(response.status)
                    this.resetPasswordVisibility = false;
                    this.resetUserEmail = '';
                    this.errorResetVisibility = true; // показать окно с ошибкой восстановления
                } else {
                    console.log(response.status)
                    this.resetPasswordVisibility = false;
                    this.resetUserEmail = '';
                    this.errorResetVisibility = true; // показать окно с ошибкой восстановления
                }
            }).catch((error) => {
                console.log(error);
                this.resetPasswordVisibility = false;
                this.resetUserEmail = '';
                this.errorResetVisibility = true; // показать окно с ошибкой восстановления
            });
        },
        drawErrorLogIn() {
            this.loginFormVisibility = false;
            this.errorLoginVisibility = true;
        },
        tryLogIn() {
            this.loginFormVisibility = true;
            this.errorLoginVisibility = false;
        },
        closeRegMessage() {
            this.regMessageVisibility = false;
            this.loginFormVisibility = true;
            this.regFormVisibility = false;
        },
        tryRegWindow() {
            this.errorMessageVisibility = false;
            this.regFormVisibility = true;
        },
        logIn() {
            if (this.userEmail == undefined || this.userEmail.trim().length === 0) {
                this.userEmailIsEmpty = true;
            }
            else {
                this.userEmailIsEmpty = false;
            }
            if (this.userPass == undefined || this.userPass.trim().length === 0) {
                this.userPassIsEmpty = true;
            }
            else {
                this.userPassIsEmpty = false;
            }
            if (!this.userEmailIsEmpty && !this.userPassIsEmpty) {

                // проверка полей to do
                axios.post('http://cloud.io-tech.ru/api/auth/token/login/', {
                    "password": this.userPass,
                    "email": this.userEmail
                }, {
                    headers: {
                        'Content-Type': 'application/json; charset=utf-8'
                    }
                }).then((response) => {
                    if (response.status === 200) {
                        sessionStorage.setItem('token', response.data.auth_token);
                        this.drawLogIn(); // если успешно
                    } else if (response.status === 400) {
                        this.drawErrorLogIn();
                    } else {
                        this.drawErrorLogIn();
                    }
                }).catch((error) => {
                    this.errorLoginMessage = `Ошибка ${error.response.status}`;
                    this.drawErrorLogIn();
                });
            }
        },
        logOut() {
            // удалить токен текущего пользователя
            sessionStorage.removeItem('token');

            this.controllerList = [];

            this.logInIndication = false;
            clearTimeout(this.getMainData); // остановить запрос по всем устройствам

            this.userEmail = '';
            this.userPass = '';
            this.mainPageVisibility = false;
            this.loginPageVisibility = true;

            for (let page in this.pages) {
                this.pages[page] = false;
            }

            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        drawLogIn() {
            this.mainPageVisibility = true;
            this.loginPageVisibility = false;

            // вывести в верхнее меню ИмяФамилию и роль в проекте
            axios.get('http://cloud.io-tech.ru/api/users/me/',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.saveUserData = response.data;

                    this.getMainData(); // запустить функцию, запрос на все устройства

                    if (this.saveUserData.profile.role === 'User') {
                        this.access = false;
                        this.saveUserData.profile.role = 'наблюдатель';
                        this.listUsersBtnVisibility = false;
                        this.projBtnVisibility = false;
                    } else if (this.saveUserData.profile.role === 'Admin') {
                        this.saveUserData.profile.role = 'администратор';
                        this.access = true;
                        this.listUsersBtnVisibility = true;
                        this.projBtnVisibility = true;
                    } else if (this.saveUserData.profile.role === 'Moderator') {
                        this.saveUserData.profile.role = 'модератор';
                        this.access = false;
                        this.listUsersBtnVisibility = false;
                        this.projBtnVisibility = false;
                    }

                    this.dateFormatting();
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                })
        },
        getMainData() {
            axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.controllerList = response.data.results;

                    this.controllerList.forEach(el => {
                        if (el.status === null) {
                            el.status = {
                                created_at: '-',
                                bat_v: '-',
                                dbi: '-',
                                event_code: '-'
                            }
                        } else {
                            let date = el.status.measured_at;
                            if (date !== undefined) {
                                if (date !== null) {
                                    let formatDate = date.split(',');
                                    el.status.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
                                }
                            }
                        }
                    })

                    this.controllerList.sort(function (a, b) { // сортировка controllerList по id
                        if (a.id > b.id) {
                            return 1;
                        }
                        if (a.id < b.id) {
                            return -1;
                        }
                        return 0;
                    });

                    if (!this.logInIndication) {
                        this.pages.mapPageVisibility = true; // показать карту (только первый раз);
                        this.logInIndication = true;
                    }

                    if (this.mainPageVisibility) { // запустить таймаут пока пользователь авторизован, каждые 5 минут запрос на устройства
                        setTimeout(this.getMainData, 300000);
                    }

                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        dateFormatting() {
            // удаление запятой в датах
            this.saveUserData.profile.created = this.saveUserData.profile.created.replace(',', ' ');
            this.saveUserData.profile.updated = this.saveUserData.profile.updated.replace(',', ' ');
        },
        sidenavToggle() {
            this.sidenavIsHidden = !this.sidenavIsHidden;
            this.contentIsCompressed = !this.contentIsCompressed;
        },
        openExtraMenu() {
            this.extraMenuIsHidden = !this.extraMenuIsHidden;
        },
        openPersonalArea() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.personalAreaPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openUserListPage() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.listUsersPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openMap() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.mapPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openControl() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.controlPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openProject() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.projectPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openList() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.listPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openSubscription() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.subscriptionPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openEvents() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.eventsPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openCommands() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.commandsPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openReports() {
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.reportsPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openMainControllerPage(id) { // открыть отдельную страницу с контроллером
            this.controllerId = id;
            for (let page in this.pages) {
                this.pages[page] = false;
            }
            this.pages.controllerPageVisibility = true;
            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        editAuthorizedUser(data) {
            this.saveUserData = data;

            if (this.saveUserData.profile.role === 'User') {
                this.saveUserData.profile.role = 'наблюдатель';
            } else if (this.saveUserData.profile.role === 'Admin') {
                this.saveUserData.profile.role = 'администратор';
            } else if (this.saveUserData.profile.role === 'Moderator') {
                this.saveUserData.profile.role = 'модератор';
            }

            this.dateFormatting();
        },
        deleteControllerFromMainList(id) {
            this.controllerList = this.controllerList.filter((controller) => controller.id !== id);
        },
        addControllerToMainList() {
            clearTimeout(this.getMainData); // остановить запрос по всем устройствам
            this.getMainData();
        },
        deleteControllerFromList(id) { // удаление контроллера (после нажатия на кнопку «Удалить контроллер» из страницы конкретного контроллера)
            this.controllerList = this.controllerList.filter((controller) => controller.id !== id);
            this.openList();
        }
    }
}
</script>

<style>
/* authorization */

.logo-container {
    text-align: center;
}

.content img {
    margin-bottom: 50px;
    width: 100%;
}

.form input {
    /* padding: 14px 16px; */
    padding: 10px 12px;
    border-radius: 8px;
    background-color: rgba(248, 246, 244, 0.3);
    color: #ffffff;
    font-size: 14px;
    margin-bottom: 8px;
}

.form input:focus {
    background-color: rgba(248, 246, 244, 0.42);
}

input::placeholder {
    color: #dad7d4;
}

.last-input {
    margin-bottom: 10px !important;
}

.form {
    display: flex;
    flex-direction: column;
    width: 21%;
}

.form button {
    font-weight: 400;
    font-size: 14px;
}

.form input:-webkit-autofill {
    -webkit-box-shadow: 0 0 0 30px #f8f6f4 inset;
    /* цвет фона */
    -webkit-text-fill-color: #0E1626;
    /* цвет текста */
}

.login-btn {
    padding: 10px 0;
    border-radius: 8px;
    color: #0E1626;
    background-color: #F8F6F4;
    margin-bottom: 8px;
}

.login-btn:hover {
    box-shadow: 0 0 8px 0 rgba(41, 59, 95, 0.5);
}

.form-btns__container {
    display: flex;
    justify-content: space-between;
}

.reg-btn {
    background-color: transparent;
    color: #F8F6F4;
    padding: 4px 2px;
    border-radius: 8px;
    text-align: right;
    font-size: 12px !important;
}

.reg-btn_back {
    text-align: center;
}

.recover-password {
    background-color: transparent;
    color: #F8F6F4;
    padding: 4px 0;
    text-align: left;
    font-size: 12px !important;
    border-radius: 8px;
}

.empty_input {
    outline: 2px solid #F4CA8D;
}

.personalarea_active {
    background-color: #293B5F !important;
    color: #F8F6F4 !important;
}

.icon-logout {
    background-image: url(../img/logout.svg);
}

.icon-personal-area {
    background-image: url(../img/pa.svg);
}

#button-logout:hover .icon-logout {
    background-image: url(../img/logout-hover.svg);
}

#button-pa:hover .icon-personal-area,
#sidenav__userlist:hover .icon-personal-area,
.icon_personal_area_active {
    background-image: url(../img/pa-hover.svg);
}

.reg-title {
    text-align: center;
    color: white;
    margin-bottom: 56px;
    font-size: 20px;
}

/* структура страниц */

.page-content__container {
    padding: 80px 24px 24px 24px;
    background-color: #EEEEEE;
    position: relative;
}

.page-content__title p,
.controller-page__title {
    margin: 0 0 24px 0;
    font-weight: 500;
    font-size: 18px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #0e1626;
}

.page-content__title div,
.account-separator {
    height: 1px;
    background: rgba(0, 0, 0, 0.1);
    margin-bottom: 24px;
}

/* table */

.basket {
    background-image: url(../img/account/basket.svg);
    margin-right: 8px;
}

.settings {
    background-image: url(../img/account/settings.svg);
}

.tr-name {
    margin: 16px 0 6px;
    font-weight: 500;
}

.tr-email {
    margin-bottom: 16px;
}

.tr-id {
    width: 40px;
}

.tr-id input {
    width: 70%;
    margin-left: 0px;
}
</style>