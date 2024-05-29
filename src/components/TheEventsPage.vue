<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>События</p>
            <div></div>
        </div>
        <table>
            <thead>
                <tr>
                    <th class="tr-id">
                        <!-- <input v-model.trim="searchUserID" type="number" placeholder="ID"
                            v-on:input="searchUser"> -->
                    </th>
                    <th>
                        <!-- <input v-model.trim="searchUserName" type="text" placeholder="Имя"
                            v-on:input="searchUserByName"> -->
                    </th>
                    <th>
                        <!-- <input v-model.trim="searchUserEmail" type="text" placeholder="email"
                            v-on:input="searchUserByEmail"> -->
                    </th>
                    <th class="select_role">
                        <!-- <button class="select__button" @click="selectRole()"
                            v-bind:class="{ opened_button: selectorTableVisible }">{{ selectorTableContent }}<i
                                class="delta" v-bind:class="{ inverted_delta: selectorTableVisible }"></i></button>
                        <ul class="dropdown__list_table"
                            v-bind:class="{ dropdown__list_visible: selectorTableVisible }">
                            <li class="dropdown__list-item_table" @click="selectTableAllRoles()">все роли</li>
                            <li class="dropdown__list-item_table" @click="selectTableObserver()">наблюдатель</li>
                            <li class="dropdown__list-item_table" @click="selectTableModer()">модератор</li>
                            <li class="dropdown__list-item_table" @click="selectTableAdmin()">администратор</li>
                        </ul> -->
                    </th>
                    <th class="sort-by-date">дата создания</th>
                    <th class="sort-by-date">дата обновления</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                </tr>
            </tbody>
        </table>
    </div>
</template>