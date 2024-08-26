<template>
    <div class="window-container">
        <div class="window" v-if="addWindow">
            <h3>Добавление контроллера</h3>
            <div class="dropdown">
                <button class="dropdown__button" @click="chooseContrType()"
                    v-bind:class="{ opened_button: selectorVisible }">{{ selectorContent }}<i class="white_delta"
                        v-bind:class="{ inverted_white_delta: selectorVisible }"></i></button>
                <ul class="dropdown__list" v-bind:class="{ dropdown__list_visible: selectorVisible }">
                    <div></div>
                    <li class="dropdown__list-item" v-for="item in deviceTypesStorage" :key="item"
                        @click="chooseThisType(item)">{{ item.device_type_name }}</li>
                </ul>
                <input type="text" v-model.trim="newContrName" placeholder="Название">
                <input type="text" v-model.trim="newContrDesc" placeholder="Описание">
                <input type="text" v-model.trim="newContrSn" placeholder="Серийный номер">
                <input type="number" v-model.trim="newContrPin" placeholder="Пин-код">
                <div class="btn-container">
                    <button className="save-btn" @click="sendNewController()">Сохранить</button>
                    <button className="cancel-btn" @click="closeAddWindow">Отменить</button>
                </div>
            </div>
        </div>
        <div class="window" v-if="contrAddedSuccessfully">
            <h3>Контроллер добавлен</h3>
            <div class="btn-container">
                <button className="save-btn" @click="openAddWindow()">Добавить ещё</button>
                <button className="cancel-btn" @click="closeAddWindow">Выход</button>
            </div>
        </div>
        <div class="window" v-if="contrAddedError">
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
            selectorContent: 'Тип устройства',
            numbSelectorContent: null, // тип устройства по умолчанию
            selectorVisible: false,

            // v-if
            contrAddedSuccessfully: false,
            addWindow: true,
            contrAddedError: false,

            addErrorText: 'Ошибка',

            newContrPin: '',
            newContrSn: '',
            newContrName: '',
            newContrDesc: '',

            serviseComp: null,
            deviceTypesStorage: []
        }
    },
    props: {
        saveUserData: Object
    },
    methods: {
        chooseContrType() {
            this.selectorVisible = !this.selectorVisible;
        },
        chooseThisType(item) {
            this.selectorContent = item.device_type_name;
            this.numbSelectorContent = item.id; // выбор из списка
            this.selectorVisible = false;
        },
        sendNewController() {
            axios.post('http://cloud.io-tech.ru/api/devices/',
                {
                    "name": this.newContrName,
                    "description": this.newContrDesc,
                    "sn": this.newContrSn,
                    "pin": this.newContrPin,
                    "device_type_name": this.numbSelectorContent,
                    "account": this.saveUserData.profile.account,
                    "installer": this.serviseComp
                }, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 201) { // контроллер успешно добавлен

                    this.$emit('newControllerLine');

                    this.newContrSn = '';
                    this.newContrPin = '';
                    this.newContrName = '';
                    this.newContrDesc = '';

                    this.numbSelectorContent = null; // тип по умолчанию
                    this.selectorContent = 'Тип устройства';

                    this.selectorVisible = false;

                    // сообщение об успешном выводе

                    this.contrAddedSuccessfully = true;
                    this.addWindow = false;

                } else {
                    this.errorWindow();
                }
            }).catch((error) => {
                this.addErrorText = error.request.responseText;
                this.errorWindow();
            });
        },
        errorWindow() {
            this.contrAddedError = true;
            this.addWindow = false;
        },
        closeAddWindow() {
            this.$emit('closeAddWindow');
            // очистить поля все
            this.newContrSn = '';
            this.newContrPin = '';
            this.newContrName = '';
            this.newContrDesc = '';

            this.numbSelectorContent = null;
            this.selectorContent = 'Тип устройства';

            this.selectorVisible = false;
            this.contrAddedError = false;

            document.getElementById('page-content').classList.remove('overflow_hidden');
        },
        openAddWindow() {
            this.contrAddedSuccessfully = false;
            this.addWindow = true;
        }
    },
    mounted() {
        axios.get('http://cloud.io-tech.ru/api/device_type_names/', // Список типов устройств
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.deviceTypesStorage = response.data;
                this.numbSelectorContent = this.deviceTypesStorage[0].id; // по умолчанию первый тип
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        axios.get('http://cloud.io-tech.ru/api/installers/', // ID аккаунта сервисной компании
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.serviseComp = response.data[0].id;
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

    }
}
</script>