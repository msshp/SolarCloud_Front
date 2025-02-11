<template>
    <!-- Страница авторизации -->
    <div v-if="loginPageVisibility" class="content"
        style="width: 100%;height: 100%; vertical-align: center; position: fixed;">
        <!-- Вход в учётную запись, если уже есть аккаунт -->
        <div v-if="loginFormVisibility" class="form">
            <div className="logo-container"><img src="../public/img/logo/logo_solar_white_rus_crop.svg" alt="logo">
            </div>
            <!-- Email -->
            <input type="email" v-model.trim="userEmail" v-bind:class="{ empty_input: userEmailIsEmpty }"
                placeholder="Email" autocomplete="current-password" required>
            <!-- Пароль -->
            <input class="last-input" type="password" v-model.trim="userPass"
                v-bind:class="{ empty_input: userPassIsEmpty }" placeholder="Пароль" autocomplete="current-password"
                required>
            <button className="login-btn" @click="logIn()">Войти</button>
            <div class="form-btns__container"><button className="recover-password"
                    @click="RecoverPassword()">Восстановить пароль</button><button className="reg-btn"
                    @click="logRegToggle()">Создать проект</button>
            </div>
        </div>
        <!-- Окно для вывода ошибок при попытке авторизации -->
        <div v-if="errorLoginVisibility" class="form">
            <p class="reg-title">{{ errorLoginMessage }}</p>
            <button v-if="errorBtnVisibility" className="login-btn" @click="tryLogIn()">Попробовать снова</button>
        </div>
        <!-- Окно для регистрации проекта -->
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
        <!-- Окно для восстановления пароля, если аккаунт уже зарегистрирован, ввод электронной почты -->
        <div v-if="resetPasswordVisibility" class="form">
            <p class="reset-p">Введите электронный адрес</p>
            <input type="email" v-model.trim="resetUserEmail" v-bind:class="{ empty_input: resetUserEmailIsEmpty }"
                placeholder="Email">
            <button className="login-btn" @click="sendEmail()">Отправить</button>
            <button className="reg-btn reg-btn_back" @click="resetBack()">Назад</button>
        </div>
        <!-- Окно для сообщения об успешной регистрации -->
        <div v-if="regMessageVisibility" class="form">
            <p class="reg-title">Вы зарегистрированы</p>
            <button className="login-btn" @click="closeRegMessage()">Войти</button>
        </div>
        <!-- Окно для вывода ошибок при попытке регистрации проекта -->
        <div v-if="errorMessageVisibility" class="form">
            <p class="reg-title">Ошибка</p>
            <button className="login-btn" @click="tryRegWindow()">Попробовать снова</button>
        </div>
        <!-- Окно для вывода ошибок при попытке восстановления пароля (вводе электронной почты) -->
        <div v-if="errorResetVisibility" class="form">
            <p class="reg-title">Ошибка</p>
            <button className="login-btn" @click="RecoverPassword()">Попробовать снова</button>
        </div>
    </div>
    <!-- Основная страница приложения (доступна авторизованным пользователям) -->
    <div v-if="mainPageVisibility">
        <div className="wrapper">
            <!-- Боковое меню -->
            <div class="sidenav" v-bind:class="{ sidenav_hidden: sidenavIsHidden }">
                <!-- Кнопка для скрытия/раскрытия бокового меню -->
                <div className="side-menu__btn" @click="sidenavToggle()">
                    <div className="icon-burger"></div>
                </div>
                <!-- Стрелка «Назад», доступна на экране только если пользователь находится на странице устройста. Ведёт в список всех устройств. -->
                <div v-if="pages.controllerPageVisibility" @click="openList()" className="side-menu__btn-back"></div>
                <div className="sidenav__top-block">
                    <img src="../public/img/logo/logo_solar_blue_rus_crop.svg" alt="logo">
                </div>
                <!-- Навигация по страницам приложения -->
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
                    <!-- Страница проекта доступна только пользователям в роли администратора/модератора -->
                    <button v-if="projBtnVisibility" id="sidenav__project" @click="openProject()"
                        v-bind:class="{ sidebtn_active: pages.projectPageVisibility }">
                        <div class="icon-project" v-bind:class="{ iconproj_active: pages.projectPageVisibility }"></div>
                        <span>Проект</span>
                    </button>
                    <!-- Список пользователей доступен только пользователям в роли администратора/модератора -->
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
                    <div class="separator"></div>
                    <button id="sidenav__events" @click="openEvents()"
                        v-bind:class="{ sidebtn_active: pages.eventsPageVisibility }">
                        <div class="icon-events" v-bind:class="{ iconevents_active: pages.eventsPageVisibility }"></div>
                        <span>События</span>
                    </button>
                    <!-- Кнопка выхода из бокового меню доступна только в мобильной версии. В десктопной версии выход через раскрывающееся меню в правом верхнем углу экрана -->
                    <button id="sidenav__sidelogout" @click="logOut()">
                        <div class="icon-logout"></div><span>Выход</span>
                    </button>
                </nav>
            </div>
            <!-- Основная страница -->
            <div id="page-content" class="page-content" v-bind:class="{ content_compressed: contentIsCompressed }">
                <!-- Верхнее меню -->
                <nav class="top-menu" v-bind:class="{ top_menu_compressed: contentIsCompressed }">
                    <!-- Раскрывающийся список в правом верхнем углу экрана (Имя Фамилия пользователя). Цвет блока меняется со белого на синий, если пользовател находится в личном кабинете. -->
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
                            <!-- Иконка «стрелка вниз/наверх» -->
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
                <!-- Контрольная панель -->
                <TheControlPage v-if="pages.controlPageVisibility" />
                <!-- Страница проекта (доступна только пользователям в роли администратора/модератора) -->
                <TheProjectPage v-if="pages.projectPageVisibility" />
                <!-- Список пользователей доступен только пользователям в роли администратора/модератора -->
                <TheUserListPage v-if="pages.listUsersPageVisibility" :saveUserData="saveUserData" :access="access" />
                <!-- Карта -->
                <TheMapPage v-if="pages.mapPageVisibility" :saveUserData="saveUserData" :controllerList="controllerList"
                    @openMainControllerPage="openMainControllerPage" />
                <!-- Список устройств -->
                <TheListPage v-if="pages.listPageVisibility" @openMainControllerPage="openMainControllerPage"
                    :access="access" :saveUserData="saveUserData" :controllerList="controllerList"
                    @deleteControllerFromMainList="deleteControllerFromMainList"
                    @addControllerToMainList="addControllerToMainList" />
                <!-- Личный кабинет -->
                <ThePersonalArea v-if="pages.personalAreaPageVisibility" :saveUserData="saveUserData"
                    @editAuthorizedUser="editAuthorizedUser" />
                <!-- Журнал событий -->
                <TheEventsPage v-if="pages.eventsPageVisibility" />
                <!-- Страница устройства -->
                <TheControllerPage v-if="pages.controllerPageVisibility" :controllerId="controllerId" :access="access"
                    @deleteControllerFromList="deleteControllerFromList" />
            </div>
        </div>
    </div>
</template>

<script>

// Список импортов компонентов в рамках приложения.

import TheMapPage from './components/TheMapPage.vue'
import ThePersonalArea from './components/ThePersonalArea.vue'
import TheProjectPage from './components/TheProjectPage.vue'
import TheUserListPage from './components/pages/TheUserListPage.vue'
import TheControlPage from './components/TheControlPage.vue'
import TheListPage from './components/TheListPage.vue'
import TheEventsPage from './components/TheEventsPage.vue'
import TheControllerPage from './components/TheControllerPage.vue'


import axios from 'axios'; // Импорт библиотеки axios для выполнения HTTP-запросов.

export default {
    components: { // Список дочерних компонентов
        TheMapPage,
        ThePersonalArea,
        TheProjectPage,
        TheUserListPage,
        TheControlPage,
        TheListPage,
        TheEventsPage,
        TheControllerPage
    },
    data() {
        return {
            // Состояния интерфейса: переменные для управления видимостью различных форм и сообщений об ошибках на экране авторизации
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

            // Переменные для хранения информации о пользователе при авторизации / регистрации проекта / восстановлении пароля
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

            pages: { // Свойства для управления видимостью различных страниц внутри приложения
                personalAreaPageVisibility: false,
                mapPageVisibility: false,
                projectPageVisibility: false,
                controlPageVisibility: false,
                listPageVisibility: false,
                eventsPageVisibility: false,
                listUsersPageVisibility: false,
                controllerPageVisibility: false,
            },

            saveUserData: { // Хранение данных пользователя (заполняется сразу при авторизации пользователя)
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
            access: true, // Идентификатор доступа ко всем страницам (по умолчанию даёт доступ ко всем, но при авторизации наблюдателя переключается на false, скрывая некоторые страницы и элементы)
            controllerId: null, // Переменная для хранения id устройства, страница которого открыта в данный момент

            controllerList: [], // Список всех устройств (заполняется сразу при авторизации пользователя)
            logInIndication: false // Индикатор для включения карты
        };
    },
    methods: {
        RecoverPassword() { // Показать форму для восстановления пароля, скрыть форму авторизации и ошибки
            this.resetPasswordVisibility = true;
            this.loginFormVisibility = false;
            this.errorResetVisibility = false;
        },
        resetBack() { // Показать форму авторизации, скрыть форму для восстановления пароля
            this.resetPasswordVisibility = false;
            this.loginFormVisibility = true;
        },
        logRegToggle() { // Переключатель между окнами авторизации и регистрации нового проекта
            this.loginFormVisibility = !this.loginFormVisibility;
            this.regFormVisibility = !this.regFormVisibility;
        },
        createProj() { // Проверка полей перед регистрацией нового проекта

            function isEmpty(value) {// Вспомогательная функция для проверки пустоты строки
                return value === undefined || value.trim().length === 0;
            }

            const fields = [ // Массив полей и соответствующих флагов
                { name: 'projName', flag: 'projNameIsEmpty' },
                { name: 'projDescription', flag: 'projDescriptionIsEmpty' },
                { name: 'projUserName', flag: 'projUserNameIsEmpty' },
                { name: 'projUserSurname', flag: 'projUserSurnameIsEmpty' },
                { name: 'projUserEmail', flag: 'projUserEmailIsEmpty' },
                { name: 'projUserLogin', flag: 'projUserLoginIsEmpty' },
                { name: 'projUserPass', flag: 'projUserPassIsEmpty' }
            ];

            fields.forEach(field => { // Проверка каждого поля и установка соответствующих флагов
                this[field.flag] = isEmpty(this[field.name]);
            });

            if (!this.projNameIsEmpty && // Проверка, все ли поля заполнены
                !this.projDescriptionIsEmpty &&
                !this.projUserNameIsEmpty &&
                !this.projUserSurnameIsEmpty &&
                !this.projUserEmailIsEmpty &&
                !this.projUserLoginIsEmpty &&
                !this.projUserPassIsEmpty) {
                this.createProjectPost(); // Если все поля заполнены, отправить информацию
            }
        },
        createProjectPost() { // Отправка информации о новом проекте в БД
            // Функция для очистки полей
            const clearFields = () => {
                this.projName = '';
                this.projDescription = '';
                this.projUserEmail = '';
                this.projUserLogin = '';
                this.projUserName = '';
                this.projUserSurname = '';
                this.projUserPass = '';
            };

            // Отправка информации о новом проекте в БД
            axios.post('http://cloud.io-tech.ru/api/accounts/signup/', {
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
            }).then((response) => { // Обработка ответа
                if (response.status === 201) { // Если успешно, то показать сообщение об успешной регистрации и очистить поля в форме регистрации проекта
                    this.regMessageVisibility = true;
                    clearFields();
                } else { // Если не успешно, показать страницу с выводом ошибки
                    this.errorMessageVisibility = true;
                }
            }).catch((error) => { // Если не успешно, показать страницу с выводом ошибки
                console.log(error);
                this.errorMessageVisibility = true;
            });

            this.regFormVisibility = false; // Закрыть окно с формой регистрации проекта
        },
        sendEmail() { // Восстановление пароля
            // Отправка POST-запроса для сброса пароля пользователя
            axios.post('http://cloud.io-tech.ru/api/users/reset_password/', {
                // Отправляем адрес электронной почты для сброса пароля
                "email": this.resetUserEmail
            }, {
                // Указываем заголовки запроса
                headers: {
                    'Content-Type': 'application/json' // Указываем тип содержимого как JSON
                }
            }).then((response) => {
                this.resetPasswordVisibility = false; // Скрываем форму сброса пароля после запроса

                this.resetUserEmail = ''; // Очищаем поле ввода электронной почты

                switch (response.status) {
                    case 204:
                        this.errorBtnVisibility = false; // Скрываем кнопку ошибки
                        // Уведомляем пользователя о том, что ссылка для сброса пароля отправлена
                        this.errorLoginMessage = "Мы отправили ссылку на ваш электронный адрес. Перейдите по ней для создания нового пароля.";
                        this.drawErrorLogIn(); // Вызываем функцию для отображения сообщения
                        break;
                    case 400: // Ошибка запроса (неправильные данные)
                    case 500: // Ошибка сервера
                    default: // Обработка всех остальных случаев
                        this.errorResetVisibility = true; // Показываем окно с ошибкой восстановления
                        break;
                }
            }).catch((error) => { // Обработка ошибок, возникших во время запроса
                console.log(error); // Логируем ошибку в консоль
                this.resetPasswordVisibility = false; // Скрываем форму сброса пароля
                this.resetUserEmail = ''; // Очищаем поле ввода электронной почты
                this.errorResetVisibility = true; // Показываем окно с ошибкой восстановления
            });
        },
        drawErrorLogIn() { // Показать окно с ошибкой при попытке авторизации пользователя
            this.loginFormVisibility = false;
            this.errorLoginVisibility = true;
        },
        tryLogIn() { // Попробовать снова выполнить авторизацию (показать окно авторизации, скрыть окно ошибки)
            this.loginFormVisibility = true;
            this.errorLoginVisibility = false;
        },
        closeRegMessage() { // Скрыть окно с сообщением об успешной регистрации, показать окно авторизации
            this.regMessageVisibility = false;
            this.loginFormVisibility = true;
            this.regFormVisibility = false;
        },
        tryRegWindow() { // Скрыть окно с ошибкой при попытке регистрации проекта
            this.errorMessageVisibility = false;
            this.regFormVisibility = true;
        },
        logIn() { // Выполнить авторизацию
            const isEmpty = (value) => value === undefined || value.trim().length === 0; // Функция для проверки, является ли строка пустой или неопределенной

            // Проверка полей электронной почты и пароля
            this.userEmailIsEmpty = isEmpty(this.userEmail);
            this.userPassIsEmpty = isEmpty(this.userPass);

            if (!this.userEmailIsEmpty && !this.userPassIsEmpty) { // Если оба поля заполнены, выполняем запрос на вход
                axios.post('http://cloud.io-tech.ru/api/auth/token/login/', {
                    "password": this.userPass,
                    "email": this.userEmail
                }, {
                    headers: {
                        'Content-Type': 'application/json; charset=utf-8'
                    }
                }).then((response) => {
                    if (response.status === 200) { // Обработка ответа сервера
                        sessionStorage.setItem('token', response.data.auth_token);
                        this.drawLogIn(); // Успешный вход, открыть доступ к приложению
                    } else {
                        this.drawErrorLogIn(); // Обработка ошибок 400 и других, показать окно с ошибкой при попытке авторизации пользователя
                    }
                }).catch((error) => {
                    this.errorLoginMessage = `Ошибка ${error.response.status}`;
                    this.drawErrorLogIn();
                });
            }
        },
        logOut() { // Выход из учетной записи
            sessionStorage.removeItem('token'); // Удаляем токен из sessionStorage
            this.controllerList = []; // Очистить список всех контроллеров
            this.logInIndication = false;
            clearTimeout(this.getMainData); // Очищаем таймер получения основных данных

            // Сбрасываем значения email и пароля пользователя
            this.userEmail = '';
            this.userPass = '';

            // Устанавливаем видимость главной страницы и страницы входа
            this.mainPageVisibility = false;
            this.loginPageVisibility = true;

            Object.keys(this.pages).forEach(page => this.pages[page] = false); // Скрыть все страницы

            document.getElementById('page-content').classList.remove('overflow_hidden'); // Убираем класс, который запрещает прокрутку
        },
        drawLogIn() { // Выполнение авторизации, доступ к приложению
            this.mainPageVisibility = true;
            this.loginPageVisibility = false;

            axios.get('http://cloud.io-tech.ru/api/users/me/',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.saveUserData = response.data; // Сохранить информацию о пользователе в переменную

                    this.getMainData(); // запустить функцию, запрос на все устройства

                    // вывести в верхнее меню ИмяФамилию и роль в проекте
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
        getMainData() { // Выполнить GET-запрос для получения списка устройств
            axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` } // Установка заголовка авторизации с токеном из sessionStorage
                }).then((response) => {
                    // обработка успешного запроса
                    this.controllerList = response.data.results; // Сохранение результатов запроса в переменной

                    this.controllerList.forEach(el => { // Обновление статусов устройств
                        if (el.status === null) {
                            el.status = { // Если статус устройства не задан, инициализируем его пустыми значениями
                                created_at: '-',
                                bat_v: '-',
                                dbi: '-',
                                event_code: '-'
                            }
                        } else { // Если статус устройства задан, форматируем дату
                            let date = el.status.measured_at;
                            if (date !== undefined) {
                                if (date !== null) {
                                    let formatDate = date.split(',');
                                    el.status.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3); // Обновляем созданную дату устройства в нужном формате
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

                    if (!this.logInIndication) { // Показываем карту при первом входе
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
        dateFormatting() { // удаление запятой в датах
            this.saveUserData.profile.created = this.saveUserData.profile.created.replace(',', ' ');
            this.saveUserData.profile.updated = this.saveUserData.profile.updated.replace(',', ' ');
        },
        sidenavToggle() { // скрыть/показать боковое меню
            this.sidenavIsHidden = !this.sidenavIsHidden;
            this.contentIsCompressed = !this.contentIsCompressed;
        },
        openExtraMenu() { // скрыть/показать пункцты в верхнем меню
            this.extraMenuIsHidden = !this.extraMenuIsHidden;
        },
        openPage(pageName, id = null) { // Универсальная функция для открытия страниц
            // Скрываем все страницы
            for (let page in this.pages) {
                this.pages[page] = false;
            }

            // Устанавливаем видимость выбранной страницы на true
            this.pages[pageName] = true;

            // Устанавливаем ID контроллера, если страница открывается для конкретного контроллера
            if (id !== null) {
                this.controllerId = id; // Сохраняем идентификатор контроллера, если передан
            }

            document.getElementById('page-content').classList.remove('overflow_hidden'); // Разрешить прокрутку страницы
        },

        // Метод для открытия личного кабинета
        openPersonalArea() {
            this.openPage('personalAreaPageVisibility'); // Открываем личный кабинет
        },

        // Метод для открытия списка пользователей
        openUserListPage() {
            this.openPage('listUsersPageVisibility'); // Открываем страницу списка пользователей
        },

        // Метод для открытия карты
        openMap() {
            this.openPage('mapPageVisibility'); // Открываем страницу карты
        },

        // Метод для открытия контрольной панели
        openControl() {
            this.openPage('controlPageVisibility'); // Открываем страницу управления
        },

        // Метод для открытия страницы проекта
        openProject() {
            this.openPage('projectPageVisibility'); // Открываем страницу проекта
        },

        // Метод для открытия списка устройств
        openList() {
            this.openPage('listPageVisibility'); // Открываем страницу со списком
        },

        // Метод для открытия журнала событий
        openEvents() {
            this.openPage('eventsPageVisibility'); // Открываем страницу событий
        },

        // Метод для открытия страницы отдельного контроллера
        openMainControllerPage(id) {
            this.openPage('controllerPageVisibility', id); // Открываем страницу контроллера
        },
        editAuthorizedUser(data) { // Сохраняем обновленные данные пользователя
            this.saveUserData = data;

            if (this.saveUserData.profile.role === 'User') {
                this.saveUserData.profile.role = 'наблюдатель';
            } else if (this.saveUserData.profile.role === 'Admin') {
                this.saveUserData.profile.role = 'администратор';
            } else if (this.saveUserData.profile.role === 'Moderator') {
                this.saveUserData.profile.role = 'модератор';
            }

            this.dateFormatting(); // Форматируем дату
        },
        deleteControllerFromMainList(id) { // Удалить контроллер из общего списка (id передаётся из компонента Список устройств)
            this.controllerList = this.controllerList.filter((controller) => controller.id !== id);
        },
        addControllerToMainList() { // Добавить новый контроллер, остановить обновление данных в Списке устройств и запустить функцию обновления заново.
            clearTimeout(this.getMainData); // остановить запрос по всем устройствам
            this.getMainData();
        },
        deleteControllerFromList(id) { // Удаление контроллера (после нажатия на кнопку «Удалить контроллер» из страницы конкретного контроллера)
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