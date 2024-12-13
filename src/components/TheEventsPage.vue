<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>События</p>
            <div></div>
        </div>
        <div class="events-filters">
            <div class="events-filters">
                <div class="events-filter__contrllers">Контроллеры</div>
                <div class="controller-nav__data-filter events-filter__delta">
                    <button class="dropdown__button dropdown__button-data-filter dropdown__set "
                        @click="chooseControllers()" v-bind:class="{ opened_button: selectorController }">{{
                            selectorControllerContent
                        }}<i class="white_delta"
                            v-bind:class="{ inverted_white_delta: selectorController }"></i></button>
                    <ul class="dropdown__list dropdown__list-data-filter dropdown__list-set"
                        v-bind:class="{ dropdown__list_visible: selectorController }">
                        <div class="datafilter-separator datafilter-separator_set"></div>
                        <div class="ul-controller">
                            <li class="dropdown__list-item" @click="chooseController(this.all)">Все<div
                                    class="icon-checkbox-blank icon-checkbox_event"
                                    v-bind:class="{ manual_coords: allControllersIndicator }">
                                </div>
                            </li>
                            <li v-for="controller in controllers" :key="controller" class="dropdown__list-item"
                                @click="chooseController(controller.id, controller.name)">{{
                                    controller.name }}<div class="icon-checkbox-blank icon-checkbox_event"
                                    v-bind:class="{ manual_coords: controller.choose }">
                                </div>
                            </li>
                        </div>
                    </ul>
                </div>
            </div>
            <div class="events-filters">
                <div class="events-filter__contrllers">Тип события</div>
                <div class="controller-nav__data-filter events-filter__delta">
                    <button class="dropdown__button dropdown__button-data-filter dropdown__set" @click="chooseCType()"
                        v-bind:class="{ opened_button: selectorCType }">{{ selectorCTypeTitle }}
                        <i class="white_delta" v-bind:class="{ inverted_white_delta: selectorCType }"></i></button>
                    <ul class="dropdown__list dropdown__list-data-filter dropdown__list-set"
                        v-bind:class="{ dropdown__list_visible: selectorCType }">
                        <div class="datafilter-separator datafilter-separator_set"></div>
                        <li class="dropdown__list-item" @click="chooseControllerType(this.all)">Все<div
                                class="icon-checkbox-blank icon-checkbox_event"
                                v-bind:class="{ manual_coords: allTypesIndicator }"></div>
                        </li>
                        <li v-for="type in types" :key="type" class="dropdown__list-item"
                            @click="chooseControllerType(type.id, type.name)">{{
                                type.name }}<div class="icon-checkbox-blank icon-checkbox_event"
                                v-bind:class="{ manual_coords: type.choose }">
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
            <div class="data-filter__container">
                <div className="datetext">27.06 – 27.06</div>
                <div class="controller-nav__data-filter">
                    <button class="dropdown__button dropdown__button-data-filter dropdown__set dropdown__set-date"
                        @click="chooseTime()" v-bind:class="{ opened_button: selectortimeVisible }">{{
                            selectortimeContent }}<i class="white_delta white_delta-e"
                            v-bind:class="{ inverted_white_delta: selectortimeVisible }">
                        </i></button>
                    <ul class="dropdown__list dropdown__list-data-filter dropdown__list-set ul_date"
                        v-bind:class="{ dropdown__list_visible: selectortimeVisible }">
                        <div class="datafilter-separator datafilter-separator_set"></div>
                        <li class="dropdown__list-item" @click="showDay()">Сегодня</li>
                        <li class="dropdown__list-item" @click="showYesterday()">Последние 2 дня</li>
                        <li class="dropdown__list-item" @click="showWeek()">Последние 7 дней</li>
                        <li class="dropdown__list-item" @click="showMonth()">Последние 30 дней</li>
                        <li class="dropdown__list-item" @click="showAllTime()">Всё время</li>
                    </ul>
                </div>
            </div>

        </div>
        <div class="dashboard-table dashboard-table__events">
            <div v-if="thereIsEventsTable" class="there-is-data there-is-nodata">{{ thereIsEventsTableText }}</div>
            <div v-if="loadingEventsTable" class="there-is-data there-is-nodata loading">
                <div>
                    <div class="loader events-loader"></div>
                </div>
            </div>
            <div class="info-line info-line__title">
                <div class="c_name">Контроллер</div>
                <div class="measured_at">дата/время</div>
                <div>Код</div>
                <div class="eventstable-desc">Описание</div>
                <div class="eventstable-type">Тип</div>
            </div>
            <div class="controller-data controller-data__events">
                <div className="info-line" v-for="event in events" :key="event"
                    :style="{ backgroundColor: event.color }">
                    <div class="c_name">{{ event.device }}</div>
                    <div class="measured_at">{{ event.measured_at }}</div>
                    <div>{{ event.code }}</div>
                    <div class="eventstable-desc">{{ event.name }}</div>
                    <div class="eventstable-type">{{ event.type }}</div>
                </div>
            </div>
        </div>
        <nav class="events-pagination">
            <ul class="pagination">
                <li><button @click="prevmorePage()">
                        <div class="pagination-delta pagination-delta_more more_back"></div>
                    </button>
                    <button @click="prevPage()">
                        <div class="pagination-delta pagination-delta_back"></div>
                    </button>
                </li>
                <li class="page-btn" v-for="btn in visiblePages" :key="btn"><button
                        v-bind:class="{ pagebtn_active: btn.active }" @click="goToPage(btn)">{{
                            btn.number }}</button></li>
                <li><button @click="nextPage()">
                        <div class="pagination-delta pagination-delta_next"></div>
                    </button>
                    <button @click="nextmorePage()">
                        <div class="pagination-delta pagination-delta_more"></div>
                    </button>
                </li>
            </ul>
        </nav>
    </div>
</template>

<script>

import axios from 'axios';

export default {
    data() {
        return {
            // loading

            loadingEventsTable: true,
            // фильтр по времени
            dateEnd: '',
            dateStart: '',
            dateText: '',
            selectortimeVisible: false,
            selectortimeContent: '',

            // фильтр по контроллерам

            selectorController: false,
            selectorControllerContent: 'Все',
            controllers: [],
            allControllersIndicator: true,

            // фильтр по типам

            selectorCType: false,
            selectorCTypeTitle: 'Все',
            allTypesIndicator: true,
            selectedTypes: [0, 1, 2, 3],
            types: [
                {
                    id: 1,
                    name: 'Информация',
                    choose: true
                },
                {
                    id: 2,
                    name: 'Предупреждения',
                    choose: true
                },
                {
                    id: 3,
                    name: 'Ошибки',
                    choose: true
                }
            ],
            all: 'all',

            events: [],
            thereIsEventsTable: false,
            idsOfControllers: [],
            thereIsEventsTableText: '',

            pagesCount: 0,
            pagesCountArr: [],

            visiblePages: [],

            prev: 0,
            next: 5,
            page: 1
        }
    },
    methods: {
        chooseTime() {
            this.selectortimeVisible = !this.selectortimeVisible;
        },
        showDay() { // по умолчанию за 1 день последний
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime());
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;
            this.selectortimeContent = 'Сегодня';

            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        showYesterday() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime());
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate() - 1)}`;

            this.selectortimeContent = 'Последние 2 дня';

            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        showWeek() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime() - (7 * 24 * 60 * 60 * 1000));
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;
            this.selectortimeContent = 'Последние 7 дней';

            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        showMonth() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            let datestart = new Date(today.getTime() - (30 * 24 * 60 * 60 * 1000));
            this.dateStart = `${datestart.getFullYear()}-${this.twoDigits(datestart.getMonth() + 1)}-${this.twoDigits(datestart.getDate())}`;
            this.selectortimeContent = 'Последние 30 дней';

            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        showAllTime() {
            let today = new Date(); // сегодня
            this.dateEnd = `${today.getFullYear()}-${this.twoDigits(today.getMonth() + 1)}-${this.twoDigits(today.getDate())}`; // формат 2024-02-01
            this.dateStart = '2024-02-01';
            this.selectortimeContent = 'Всё время';

            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        twoDigits(num) {
            return ('0' + num).slice(-2);
        },
        chooseControllers() {
            this.selectorController = !this.selectorController;
        },
        chooseCType() {
            this.selectorCType = !this.selectorCType;
        },
        chooseController(id) { // фильтр по контроллеру
            if (id === 'all') {
                this.allControllersIndicator = !this.allControllersIndicator;

                if (this.allControllersIndicator === true) {
                    this.selectorControllerContent = 'Все';
                    this.controllers.forEach((el) => { // отметить все галочки
                        el.choose = true;
                    })

                    this.idsOfControllers = [];
                    this.controllers.forEach((el) => {
                        this.idsOfControllers.push(el.id);
                    });
                } else {
                    this.selectorControllerContent = 'Выбрать';
                    this.controllers.forEach((el) => { // отметить все галочки
                        el.choose = false;
                    })

                    this.idsOfControllers = [];
                }
            } else {
                const controllerToUpdate = this.controllers.find(controller => controller.id === id);

                if (controllerToUpdate) {
                    controllerToUpdate.choose = !controllerToUpdate.choose; // убрать галочку/поставить галочку

                    if (controllerToUpdate.choose === false) { // удалить id из списка с id контроллеров
                        const indexToRemove = this.idsOfControllers.indexOf(id);
                        if (indexToRemove !== -1) {
                            this.idsOfControllers.splice(indexToRemove, 1);
                        }

                        if (this.idsOfControllers.length === 0) {
                            this.selectorControllerContent = 'Выбрать';
                        } else {
                            this.selectorControllerContent = `Посмотреть выбранные`;
                        }

                        this.allControllersIndicator = false;
                    } else { // добавить в список
                        this.idsOfControllers.push(id);

                        this.selectorControllerContent = `Посмотреть выбранные`;

                        if (this.controllers.length === this.idsOfControllers.length) { // если кол-во галочек = кол-ву контроллеров, отметить все
                            this.selectorControllerContent = `Все`;
                            this.allControllersIndicator = true;
                        }
                    }
                }
            }
            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        chooseControllerType(id, name) {
            if (id === 'all') {
                this.allTypesIndicator = !this.allTypesIndicator;

                if (this.allTypesIndicator === true) {
                    this.selectorCTypeTitle = 'Все';
                    this.types.forEach((el) => { // отметить все галочки
                        el.choose = true;
                    });

                    this.selectedTypes = [];
                    this.types.forEach((el) => {
                        this.selectedTypes.push(el.id);
                    });
                } else {
                    this.selectorCTypeTitle = 'Выбрать';
                    this.types.forEach((el) => { // отметить все галочки
                        el.choose = false;
                    });

                    this.selectedTypes = [];
                }
            } else {
                this.selectorCTypeTitle = name;

                const typeToUpdate = this.types.find(type => type.id === id);

                if (typeToUpdate) {
                    typeToUpdate.choose = !typeToUpdate.choose; // убрать галочку/поставить галочку

                    if (typeToUpdate.choose === false) { // удалить id из списка с id типов
                        const indexToRemove = this.selectedTypes.indexOf(id);
                        if (indexToRemove !== -1) {
                            this.selectedTypes.splice(indexToRemove, 1);
                        }

                        if (this.selectedTypes.length === 0) {
                            this.selectorCTypeTitle = `Выбрать`;
                        } else {
                            this.selectorCTypeTitle = `Посмотреть выбранные`;
                        }
                        this.allTypesIndicator = false;
                    } else { // добавить в список
                        this.selectedTypes.push(id);

                        if (this.types.length === this.selectedTypes.length) { // если кол-во галочек = кол-ву контроллеров, отметить все
                            this.selectorCTypeTitle = `Все`;
                            this.allTypesIndicator = true;
                        } else {
                            this.selectorCTypeTitle = `Посмотреть выбранные`;
                        }
                    }
                }
            }
            this.pagesCount = 0;
            this.pagesCountArr = [];
            this.page = 1;

            this.getEvents();
        },
        getControllerList() {
            axios.get('http://cloud.io-tech.ru/api/devices/limited/?limit=10000',
                {
                    headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                }).then((response) => {
                    // обработка успешного запроса
                    this.controllers = response.data.results;

                    this.controllers.sort(function (a, b) { // сортировка controllers по id
                        if (a.id > b.id) {
                            return 1;
                        }
                        if (a.id < b.id) {
                            return -1;
                        }
                        return 0;
                    });

                    this.controllers.forEach(el => {
                        el.choose = true;
                        this.idsOfControllers.push(el.id);
                    })
                    this.showDay(); // вывести 1 день
                }).catch((error) => {
                    // обработка ошибки
                    console.log(error);
                });
        },
        getEvents() { // получить список событий (по умолчанию все приборы, все типы за последний день)
            this.thereIsEventsTable = false; // скрыть «Нет событий за период»
            this.loadingEventsTable = true; // показать loading

            if (this.pagesCount === 0) {
                this.pagesCount = 0;
                this.pagesCountArr = [];
            }

            // // закрыть все фильтры
            // this.selectorController = false;
            // this.selectorCType = false;
            // this.selectortimeVisible = false;

            if (this.idsOfControllers.length === 0 || this.selectedTypes.length === 0) {
                this.events = [];
                this.thereIsEventsTable = true; // показать блок «Нет событий за период

                if (this.idsOfControllers.length === 0) {
                    this.thereIsEventsTableText = 'Контроллеры не выбраны';
                } else if (this.selectedTypes.length === 0) {
                    this.thereIsEventsTableText = 'Типы событий не выбраны';
                }
                this.loadingEventsTable = false; // убрать loading
            } else {
                // строка из массива с контроллерами
                let idsOfControllersString = this.idsOfControllers.join(',');

                let idsOfTypesString = this.selectedTypes.join(',');

                axios.get(`http://cloud.io-tech.ru/api/events/?date_start=${this.dateStart}&device=${idsOfControllersString}&type=${idsOfTypesString}&limit=20&page=${this.page}`,
                    {
                        headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
                    }).then((response) => {
                        if (response.status === 200) {
                            this.events = response.data.results;

                            if (this.events.length === 0) {
                                this.thereIsEventsTable = true; // показать блок «Нет событий за период
                                this.thereIsEventsTableText = 'Нет событий за период';
                            } else { // обработать массив (на русский перевести тип события и задать цвет)

                                if (this.pagesCount === 0) {
                                    // выстроить пагинацию
                                    this.pagesCount = response.data.count / 20;
                                    if (this.pagesCount % 1 !== 0) { // Проверяем, есть ли у числа дробная часть
                                        this.pagesCount = Math.ceil(this.pagesCount); // 
                                    }

                                    for (let i = 1; i < this.pagesCount + 1; i++) {
                                        this.pagesCountArr.push({
                                            number: i,
                                            active: false
                                        });
                                    }

                                    this.visiblePages = this.pagesCountArr.slice(0, 5);
                                }

                                // Выделить активную кнопку в пагинации
                                const selectedPage = this.pagesCountArr.find(page => page.number === this.page);
                                // Установить свойство active на true у найденного объекта
                                selectedPage.active = true;

                                this.events.forEach((el) => {
                                    let date = el.measured_at;
                                    let formatDate = date.split(',');
                                    el.measured_at = formatDate[0] + ' ' + formatDate[1].slice(0, -3);

                                    const codeToTypeMap = {
                                        'Info': 'Информация',
                                        'Warning': 'Предупреждение',
                                        'Error': 'Ошибка'
                                    };

                                    el.type = codeToTypeMap[el.event_type];

                                    const colorToTypeMap = {
                                        'Info': '#F8F6F4',
                                        'Warning': '#F4CA8D',
                                        'Error': '#f57878'
                                    };

                                    el.color = colorToTypeMap[el.event_type];
                                })
                            }
                            this.loadingEventsTable = false; // убрать loading
                        }
                    }).catch((error) => {
                        // обработка ошибки
                        console.log(error);
                        this.thereIsEventsTable = true; // показать блок «Нет событий за период»
                        this.thereIsEventsTableText = 'Возникла ошибка';
                        this.loadingEventsTable = false; // убрать loading
                    });
            }
        },
        goToPage(page) {
            this.page = page.number;

            this.pagesCountArr.forEach(el => el.active = false);
            page.active = true;
            this.getEvents();
        },
        prevmorePage() { // переключение в пагинации (5 кнопок назад)
            if (this.prev > 0) {
                this.next = this.next - 5;
                this.prev = this.next - 5;
                this.visiblePages = this.pagesCountArr.slice(this.prev, this.next);
            }
        },
        prevPage() {
            if (this.page > 1) {
                this.pagesCountArr.forEach(el => el.active = false);
                this.page--;
                this.getEvents();
            }
        },
        nextPage() {
            if (this.page < this.pagesCount) { // это общее количество страниц
                this.pagesCountArr.forEach(el => el.active = false);
                this.page++;
                this.getEvents();
            }
        },
        nextmorePage() {
            if (this.next < this.pagesCount) {
                this.next = this.next + 5;
                this.prev = this.next - 5;
                this.visiblePages = this.pagesCountArr.slice(this.prev, this.next);
            }
        }
    },
    mounted() {
        this.getControllerList(); // получить список всех контроллеров
    }
}

</script>

<style>
.events-filters {
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.events-filters button {
    height: 32px;
    font-size: 14px;
}

.events-filter__contrllers {
    margin-right: 8px;
}

.events-filters i {
    padding: 0 12px !important;
}

.events-filters ul {
    top: 30px;
}

.events-filters ul li {
    font-size: 14px;
}

.dropdown__set-date,
.ul_date {
    width: 208px !important;
}

.dropdown__set-date i {
    right: 0 !important;
}

.dashboard-table__events {
    margin-top: 12px;
}

.ul-controller {
    max-height: 450px;
    overflow-y: scroll;
    margin: 0;
}

.there-is-nodata {
    height: 102%;
}

.icon-checkbox_event {
    border: none;
    margin: 0 4px 0 auto;
}

.events-loader {
    margin-top: 10px !important;
}

.pagination {
    display: flex;
    list-style-type: none;
    padding: 0;
    align-items: center;
    margin: 20px 0 0;
}

.pagination li {
    margin-right: 5px;
    align-items: center;
    display: flex;
}

.pagination button {
    padding: 5px 8px;
    cursor: pointer;
    border-radius: 4px;
}

.events-pagination {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.pagination-delta {
    background-image: url(../img/delta.svg);
    background-repeat: no-repeat;
    width: 17px;
    background-size: contain;
    height: 11px;
    margin: 0;
    padding: 0;
    border: 0;
}

.pagination-delta_more {
    background-image: url(../img/more.svg);
    width: 14px;
    height: 18px;
}

.more_back {
    transform: rotate(180deg);
}

.pagination-delta_back {
    transform: rotate(90deg);
}

.pagination-delta_next {
    transform: rotate(-90deg);
}

.pagebtn_active,
.page-btn button:hover {
    background-color: #293b5f;
    color: #f8f6f4;
}

.controller-data__events {
    height: 59vh !important;
}

.events-filter__delta i {
    right: -20px !important;
}

.events-filter__delta ul {
    width: 230px;
}

.white_delta-e {
    height: 32px !important;
}

.c_name {
    width: 248px !important;
    font-weight: 500 !important;
}

@media (min-width: 1500px) {
    .controller-data__events {
        height: 62vh !important;
    }
}

@media (min-width: 1600px) {
    .controller-data__events {
        height: 67vh !important;
    }
}

@media (min-width: 1900px) {
    .controller-data__events {
        height: 63vh !important;
    }
}

@media (min-width: 2000px) {
    .controller-data__events {
        height: 56vh !important;
    }
}
</style>