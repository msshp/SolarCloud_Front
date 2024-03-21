<template>
    <div class="page-content__container">
        <div class="page-content__title">
            <p>Проект</p>
            <div></div>
        </div>
        <div className="proj-info">
            <section>
                <p>Название проекта</p>
                <input type="text" v-model.trim="projName" placeholder="Название проекта">
                <p>Описание</p>
                <input type="text" v-model.trim="projDesc" placeholder="Описание проекта">
                <div><button class="save-btn save-proj-btn" @click="sendNewProj">Сохранить</button>
                    <span v-bind:class="{ saved_projinfo: savedProjinfo }" class="projedit_done">Данные обновлены</span>
                </div>
            </section>
            <section>
                <div class="proj-info__img"></div>
                <div class="proj-info__block">
                    <p class="">Количество контроллеров</p>
                    <p class="proj-info__block-num"> {{ controllersNum }}</p>
                </div>
            </section>
        </div>
    </div>
</template>

<script>

import axios from 'axios';

export default {
    data() {
        return {
            projName: '',
            projDesc: '',
            projectInfo: {},
            controllersNum: null,
            savedProjinfo: false // строчка «Данные обновлены»
        }
    },
    methods: {
        sendNewProj() {
            // отправка новых данных
            axios.patch(`http://cloud.io-tech.ru/api/accounts/${this.projectInfo.id}/`,
                {
                    "name": this.projName,
                    "description": this.projDesc
                }, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`,
                    'Content-Type': 'application/json; charset=utf-8'
                }
            }).then((response) => { // обработка ошибок
                if (response.status === 200) { // данные обновлены

                    // сообщение об успешном выводе
                    this.savedProjinfo = true;
                    setTimeout(() => {
                        this.savedProjinfo = false;
                    }, 5000)
                }
            }).catch((error) => {
                console.log(error);
            });
        }
    },
    mounted() {
        // запрос данных пользователя
        axios.get(`http://cloud.io-tech.ru/api/accounts/me/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                this.projectInfo = response.data; // сохранение данных юзера

                // заполнение полей
                this.projName = this.projectInfo.name;
                this.projDesc = this.projectInfo.description;
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });

        axios.get(`http://cloud.io-tech.ru/api/devices/limited/`,
            {
                headers: { 'Authorization': `Token ${sessionStorage.getItem('token')}` }
            }).then((response) => {
                // заполнение полей
                this.controllersNum = response.data.results.length;
            }).catch((error) => {
                // обработка ошибки
                console.log(error);
            });
    }
}</script>

<style>
@-webkit-keyframes fade-out {
    0% {
        opacity: 1;
    }

    100% {
        opacity: 0;
    }
}

@keyframes fade-out {
    0% {
        opacity: 1;
    }

    100% {
        opacity: 0;
    }
}

.proj-info {
    display: flex;
}

.proj-info section {
    height: 242px;
    width: 624px;
    background-color: #F8F6F4;
    border-radius: 8px;
    font-weight: 400;
    font-size: 14px;
    line-height: 129%;
    color: #293b5f;
    padding: 32px;
}

.proj-info section:first-child {
    margin-right: 24px;
}

.proj-info section:last-child {
    display: flex;
    align-items: center;
    justify-content: center;
}

.proj-info p {
    margin: 0;
}

.proj-info input {
    width: 100%;
    border-bottom: 1px solid #293b5f;
    height: 48px;
    background: transparent;
    font-weight: 400;
    font-size: 16px;
    line-height: 112%;
    color: #293b5f;
    margin-bottom: 32px;
}

.save-proj-btn {
    border-radius: 8px;
    width: 132px;
    height: 40px;
    font-weight: 500;
    font-size: 14px;
    line-height: 112%;
    text-align: center;
}

.projedit_done {
    margin-left: 10px;
    font-size: 14px;
    color: green;
    display: none;
}

.proj-info__img {
    background-image: url(../img/account/proj.png);
    margin: 16px 0 16px 16px;
    width: 135.6px;
    height: 135.6px;
    background-repeat: no-repeat;
    border-radius: 8px;
}

.proj-info__block {
    margin-left: 62px;
    font-size: 16px;
    text-align: center;
}

.proj-info__block-num {
    font-weight: 500;
    font-size: 48px;
    color: #0e1626;
    margin-top: 40px !important;
}

.saved_projinfo {
    display: inline;
    -webkit-animation: fade-out 5s ease-out both;
    animation: fade-out 5s ease-out both;

}
</style>