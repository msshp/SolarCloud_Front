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
                    <th class="sort-by-date">Состояние</th>
                </tr>
            </thead>
            <div v-if="list.loading" class="loading">
                <div>
                    <div class="loader"></div>
                </div>
            </div>
            <tbody class="list-tbody" v-if="list.listTable">
                <TheControllerLine :controllerList="controllerTableList"
                    @openMainControllerPage="openMainControllerPage"
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

export default {
    components: {
        TheControllerLine,
        TheAddControllerWindow
    },
    props: {
        access: Boolean,
        saveUserData: Object,
        controllerList: Array
    },
    data() {
        return {
            addControllerWindowVis: false,

            searchIdControllerById: '',
            searchValControllerByControllerName: '',
            searchValControllerBySn: '',
            controllerTableList: [],

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
        sortListByVoltageToggle() {
            this.sortlistByVoltageActive = !this.sortlistByVoltageActive;
            this.sortlistByLastTimeActive = false;

            const table = document.querySelector('.list-tbody');
            const rows = Array.from(table.querySelectorAll('tr'));

            rows.sort((a, b) => {
                const cellIndex = this.sortlistByVoltageActive ? 4 : 1;

                const cellA = a.querySelector(`td:nth-child(${cellIndex})`).textContent;
                const cellB = b.querySelector(`td:nth-child(${cellIndex})`).textContent;

                if (cellA === '-' || cellB === '-') {
                    if (cellA === '-') {
                        return 1;
                    } else if (cellB === '-') {
                        return -1;
                    }
                }

                const aValue = parseFloat(cellA);
                const bValue = parseFloat(cellB);

                return this.sortlistByVoltageActive ? bValue - aValue : aValue - bValue;
            });

            rows.forEach(row => table.appendChild(row));
        },
        sortListByLastTimeToggle() {
            this.sortlistByLastTimeActive = !this.sortlistByLastTimeActive;
            this.sortlistByVoltageActive = false;

            const table = document.querySelector('.list-tbody');
            const rows = Array.from(table.querySelectorAll('tr'));

            rows.sort((a, b) => {
                const cellIndex = this.sortlistByLastTimeActive ? 6 : 1;

                const cellA = a.querySelector(`td:nth-child(${cellIndex})`).textContent;
                const cellB = b.querySelector(`td:nth-child(${cellIndex})`).textContent;

                if (cellA === '-' || cellB === '-') {
                    if (cellA === '-') {
                        return 1;
                    } else if (cellB === '-') {
                        return -1;
                    }
                }

                if (this.sortlistByLastTimeActive) {
                    const dateA = new Date(cellA.replace(/(\d{2}).(\d{2}).(\d{4}) (\d{2}):(\d{2})/, '$3-$2-$1T$4:$5'));
                    const dateB = new Date(cellB.replace(/(\d{2}).(\d{2}).(\d{4}) (\d{2}):(\d{2})/, '$3-$2-$1T$4:$5'));

                    return dateB - dateA;
                } else {
                    const aValue = parseFloat(cellA);
                    const bValue = parseFloat(cellB);

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
        addNewControllerToList(newobj) {
            this.$emit('addControllerToMainList');
            this.controllerTableList.push(newobj);
        },
        getControllerList() {
            this.controllerTableList = this.controllerList;
            this.controllerTableList.sort(function (a, b) { // сортировка controllerList по id
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
        },
        deleteControllerFromList(id) {
            this.controllerTableList = this.controllerList.filter((controller) => controller.id !== id);
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