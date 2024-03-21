<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Личный кабинет</p>
            <div></div>
        </div>
        <table>
            <thead>
                <tr class="pa__titles">
                    <th class="tr-id">ID</th>
                    <th class="tr-pa">Имя</th>
                    <th class="tr-pa">email</th>
                    <th class="tr-role">роль</th>
                    <th class="tr-pa">дата создания</th>
                    <th class="tr-pa">последний сеанс</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                <tr class="pa__line">
                    <td class="tr-id">{{ saveUserData.id }}</td>
                    <td class="tr-pa"> {{ saveUserData.first_name + ' ' + saveUserData.last_name }}</td>
                    <td class="tr-pa">{{ saveUserData.email }}</td>
                    <td class="tr-role">{{ saveUserData.profile.role }}</td>
                    <td class="tr-pa">{{ saveUserData.profile.created }}</td>
                    <td class="tr-pa">{{ saveUserData.profile.updated }}</td>
                    <td class="tr-btns">
                        <!-- <button class="basket"></button><button class="settings"></button> -->
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>

export default {
    props: {
        saveUserData: Object
    }
}
</script>

<style>
.pa__titles th {
    color: rgba(14, 22, 38, 0.5);
    padding: 0;
}

.pa__line td {
    padding: 0px;
}

/* .tr-pa {
    width: 19%;
}

.tr-role {
    width: 12%;
} */
</style>