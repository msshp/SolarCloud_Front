<template>
    <div class="page-content__container">
        <div className="page-content__title page-content__title_list">
            <p>Список контроллеров</p>
            <!-- <div>
                <button class="save-exel">Сохранить в Exel</button>
                <button class="save-btn">Распечатать</button>
            </div> -->
        </div>
        <div class="account-separator"></div>
        <table>
            <thead>
                <tr>
                    <th class="tr-id"><input v-model.trim="searchIdControllerById" type="number" placeholder="ID"
                            v-on:input="searchControllerSyId">
                    </th>
                    <th class="tr-sn"><input v-model.trim="searchValControllerBySn" type="number"
                            placeholder="Серийный номер" v-on:input="searchControllerBySn">
                    </th>
                    <th><input v-model.trim="searchValControllerByControllerName" type="text" placeholder="Название"
                            v-on:input="searchControllerByControllerName"></th>
                    <th class="sort-by-date">Напряжение АКБ</th>
                    <th class="sort-by-date">Уровень сигнала GSM</th>
                    <th class="sort-by-date">Выход на связь</th>
                    <th class="sort-by-date">event</th>
                </tr>
            </thead>
            <tbody>
                <TheControllerLine :controllerList="controllerList" @openMainControllerPage="openMainControllerPage" />
            </tbody>
        </table>
    </div>
</template>

<script>

import TheControllerLine from './TheControllerLine.vue'
import axios from 'axios';

export default {
    components: {
        TheControllerLine
    },
    data() {
        return {
            searchIdControllerById: '',
            searchValControllerByControllerName: '',
            searchValControllerBySn: '',
            controllerList: []
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
        }
    },
    // вывести список контроллеров
    mounted() {
        axios.get('http://cloud.io-tech.ru/api/devices/limited/',
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // обработка успешного запроса
                this.controllerList = response.data.results;

                this.controllerList.forEach(el => {
                    let date = el.status.created_at;
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
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    }
}
</script>

<style>
.page-content__title_list {
    display: flex;
    justify-content: space-between;
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
</style>