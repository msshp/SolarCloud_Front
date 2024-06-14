<template>
    <div class="page-content__container">
        <div className="page-content__title page-content__title_list">
            <p>Список контроллеров</p>
            <button v-if="access" class="account__add-users" @click="addController()"><span>+</span> Добавить
                контроллер</button>
        </div>
        <div class="account-separator"></div>
        <table>
            <thead>
                <tr class="list-tr">
                    <th class="tr-id"><input v-model.trim="searchIdControllerById" type="number" placeholder="ID"
                            v-on:input="searchControllerSyId">
                    </th>
                    <th class="tr-sn"><input v-model.trim="searchValControllerBySn" type="number"
                            placeholder="Серийный номер" v-on:input="searchControllerBySn">
                    </th>
                    <th><input v-model.trim="searchValControllerByControllerName" type="text" placeholder="Название"
                            v-on:input="searchControllerByControllerName"></th>
                    <th class="sort-by-date">
                        <div class="sortlist-by-voltage_container">Напряжение АКБ (В)<div
                                @click="sortListByVoltageToggle()" class="sortlist-by-voltage"
                                v-bind:class="{ sortlistbyvoltage_active: sortlistByVoltageActive }"></div>
                        </div>
                    </th>
                    <th class="sort-by-date">Уровень сигнала GSM</th>
                    <th class="sort-by-date">
                        <div class="sortlist-by-voltage_container">Выход на связь<div
                                @click="sortListByLastTimeToggle()" class="sortlist-by-voltage"
                                v-bind:class="{ sortlistbyvoltage_active: sortlistByLastTimeActive }"></div>
                        </div>
                    </th>
                    <th class="sort-by-date">Ошибки</th>
                </tr>
            </thead>
            <div v-if="list.loading" class="loading">
                <div>
                    <div class="loader"></div>
                </div>
            </div>
            <tbody class="list-tbody" v-if="list.listTable">
                <TheControllerLine :controllerList="controllerList" @openMainControllerPage="openMainControllerPage"
                    @deleteControllerFromList="deleteControllerFromList" :access="access" />
            </tbody>
        </table>
    </div>
    <TheAddControllerWindow v-if="addControllerWindowVis" @closeAddWindow="closeAddWindow"
        @newControllerLine="addNewControllerToList" :saveUserData="saveUserData" />
</template>

<script>

import TheControllerLine from './TheControllerLine.vue'
import TheAddControllerWindow from './TheControllerListComponents/TheAddControllerWindow.vue'
import axios from 'axios';

export default {
    components: {
        TheControllerLine,
        TheAddControllerWindow
    },
    props: {
        access: Boolean,
        saveUserData: Object
    },
    data() {
        return {
            addControllerWindowVis: false,

            searchIdControllerById: '',
            searchValControllerByControllerName: '',
            searchValControllerBySn: '',
            controllerList: [],

            list: {
                loading: true,
                listTable: false
            },

            sortlistByVoltageActive: false, // Сортировка по напряжению
            sortlistByLastTimeActive: false // Сортировка по последнему выходу на связь
        }
    },
    methods: {
        searchControllerSyId() { // поиск устройства по ID
            let list = document.querySelectorAll('tbody tr');

            if (this.searchIdControllerById != '') {
                list.forEach(elem => {
                    if (elem.firstChild.innerText.search(this.searchIdControllerById) === -1) {
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
        searchControllerBySn() { // поиск устройства по Серийнику

            let list = document.querySelectorAll('tbody tr');

            if (this.searchValControllerBySn != '') {
                list.forEach(elem => {
                    if (elem.children[1].innerText.search(this.searchValControllerBySn) === -1) {
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
        searchControllerByControllerName() { // поиск устройства по Серийнику
            let list = document.querySelectorAll('tbody tr');

            if (this.searchValControllerByControllerName != '') {
                list.forEach(elem => {
                    if (elem.children[2].innerText.search(this.searchValControllerByControllerName) === -1) {
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
        openMainControllerPage(id) {
            this.$emit('openMainControllerPage', id);
        },
        sortListByVoltageToggle() { // сортировка по напряжению акб
            this.sortlistByVoltageActive = !this.sortlistByVoltageActive; // цвет иконки сортировки
            this.sortlistByLastTimeActive = false; // цвет иконки сортировки

            const table = document.querySelector('.list-tbody');
            const rows = Array.from(table.querySelectorAll('tr'));

            rows.sort((a, b) => {
                const cellIndex = this.sortlistByVoltageActive ? 4 : 1;

                const aValue = parseFloat(a.querySelector(`td:nth-child(${cellIndex})`).textContent);
                const bValue = parseFloat(b.querySelector(`td:nth-child(${cellIndex})`).textContent);

                return this.sortlistByVoltageActive ? bValue - aValue : aValue - bValue;
            });

            rows.forEach(row => table.appendChild(row));
        },
        sortListByLastTimeToggle() { // сотировка по последнему выходу на связь

            this.sortlistByLastTimeActive = !this.sortlistByLastTimeActive;
            this.sortlistByVoltageActive = false;

            const table = document.querySelector('.list-tbody');
            const rows = Array.from(table.querySelectorAll('tr'));

            rows.sort((a, b) => {
                const cellIndex = this.sortlistByLastTimeActive ? 6 : 1;

                if (this.sortlistByLastTimeActive) {
                    const dateA = new Date(a.querySelector(`td:nth-child(${cellIndex})`).textContent.replace(/(\d{2}).(\d{2}).(\d{4}) (\d{2}):(\d{2})/, '$3-$2-$1T$4:$5'));
                    const dateB = new Date(b.querySelector(`td:nth-child(${cellIndex})`).textContent.replace(/(\d{2}).(\d{2}).(\d{4}) (\d{2}):(\d{2})/, '$3-$2-$1T$4:$5'));

                    return dateB - dateA;
                } else {
                    const aValue = parseFloat(a.querySelector(`td:nth-child(${cellIndex})`).textContent);
                    const bValue = parseFloat(b.querySelector(`td:nth-child(${cellIndex})`).textContent);

                    return aValue - bValue;
                }
            });

            rows.forEach(row => table.appendChild(row));
        },
        addController() {
            this.addControllerWindowVis = true;
            document.getElementById('page-content').classList.add('overflow_hidden');
        },
        closeAddWindow() {
            this.addControllerWindowVis = false; // закрытие окна «Добавление пользователя»
        },
        addNewControllerToList() {
            this.getControllerList();
        },
        getControllerList() {
            axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.controllerList = response.data.results;

                    this.controllerList.forEach(el => {
                        let date = el.status.last_session;
                        if (date !== null) {
                            let formatDate = date.split(',');
                            el.status.created_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);
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

                    this.list.loading = false;
                    this.list.listTable = true;
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        deleteControllerFromList(id) {
            this.controllerList = this.controllerList.filter((controller) => controller.id !== id);
        }
    },
    mounted() {
        this.getControllerList(); // вывести список контроллеров
    }
}
</script>

<style>
.page-content__title_list {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
}

.page-content__title_list button,
.page-content__title_list p {
    margin-bottom: 0 !important;
}

.tr-sn input {
    margin-left: 12px;
}

tr th {
    text-align: left;
}

tr th input {
    margin-left: 12px;
}

.ymaps-2-1-79-balloon__close-button {
    height: 38px !important;
}

.sortlist-by-voltage_container {
    display: flex;
    align-items: center;
}

.sortlist-by-voltage {
    width: 30px;
    height: 30px;
    margin-left: 14px;
    background-image: url(../sorticon_unactive.svg);
    background-size: contain;
    background-repeat: no-repeat;
    cursor: pointer;
}

.sortlistbyvoltage_active {
    background-image: url(../sorticon_active.svg);
}

.controller-btn td {
    padding: 0 0 0 20px;
}

@media (max-width: 1600px) {
    .list-tr th {
        font-size: 12px;
    }

    table input,
    .select__button,
    .sort-by-creation {
        font-size: 12px;
    }

    .sortlist-by-voltage {
        width: 26px;
        height: 26px;
        margin-left: 8px;
    }
}
</style>