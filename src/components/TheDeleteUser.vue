<template>
    <div class="window-container window-delete">
        <div class="window" v-if="deleteWindow">
            <h3>Удалить пользователя?</h3>
            <div class="btn-container">
                <button className="save-btn" @click="deleteUserReq()">Да</button>
                <button className="cancel-btn" @click="closeDeleteWindow">Отмена</button>
            </div>
        </div>
        <div class="window" v-if="userDeleteError">
            <h3>{{ deleteErrorText }}</h3>
            <div class="btn-container">
                <button className="cancel-btn" @click="closeDeleteWindow">Выход</button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    props: {
        userIdDel: Text
    },
    data() {
        return {
            // v-if
            deleteWindow: true,
            userDeleteError: false,

            deleteErrorText: 'Ошибка'
        }
    },
    methods: {
        deleteUserReq() {
            axios.delete(`http://cloud.io-tech.ru/api/users/${this.userIdDel}/`, {
                headers: {
                    'Authorization': `Token ${sessionStorage.getItem('token')}`
                }
            }).then((response) => {
                if (response.status === 204) { // пользователь успешно удалён
                    console.log('204')
                    // сообщение об успешном удалении
                    this.deleteErrorText = 'Пользователь удалён';
                    this.errorWindow();
                    this.$emit('deleteUserFromUserList', this.userIdDel); // удалить объект в usersList

                } else if (response.status === 401) { // окно с ошибкой
                    this.deleteErrorText = 'Пользователь не авторизован';
                    this.errorWindow();
                } else if (response.status === 403) {
                    this.deleteErrorText = 'Недостаточно прав';
                    this.errorWindow();
                } else if (response.status === 404) {
                    this.deleteErrorText = 'Объект не найден';
                    this.errorWindow();
                } else {
                    this.errorWindow();
                }
            }).catch((error) => {
                console.log(error);
                this.errorWindow();
            })
        },
        errorWindow() {
            this.userDeleteError = true;
            this.deleteWindow = false;
        },
        closeDeleteWindow() {
            this.$emit('closeDeleteWindow', false);
            this.userDeleteError = false;
        }
    },
    mounted() {
        console.log('mount');
    }
}
</script>

<style>
.window {
    width: 590px;
    background-color: #F8F6F4;
    border-radius: 8px;
    padding: 48px;
}

.result_window {
    width: 320px;
    margin: 0 auto 24px;
    font-weight: 400;
    font-size: 16px;
    line-height: 112%;
    color: #0e1626;
}

h3 {
    font-weight: 500;
    font-size: 20px;
    line-height: 100%;
    letter-spacing: -0.02em;
    color: #0e1626;
    text-align: center;
    margin: 0px 0px 32px;
}

.dropdown {
    position: relative;
    display: flex;
    flex-direction: column;
}

.dropdown__button {
    height: 48px;
    width: 100%;
    text-align: left;
    border-radius: 8px;
    background: #294b8e;
    padding: 8px 20px 8px 16px;
    margin-top: 8px;
    margin-bottom: 8px;
    font-weight: 500;
    font-size: 16px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #f8f6f4;

    display: flex;
    justify-content: space-between;
    align-items: center;
}

.dropdown__button i {
    width: 20px;
    height: 20px;
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
}

.dropdown__list {
    display: none;
    position: absolute;
    left: 0;
    top: 48px;
    list-style-type: none;
    background: #294b8e;
    border-radius: 0 0 8px 8px;
    width: 100%;
    padding: 0 0 4px 0;
    margin: 0;
}

ul div {
    border: 0.3px solid #f8f6f4;
    margin: 8px 20px 0;
    margin-bottom: 12px;
}

.dropdown__list-item {
    text-align: left;
    border-radius: 8px;
    height: 40px;
    cursor: pointer;
    font-weight: 400;
    font-size: 16px;
    line-height: 125%;
    letter-spacing: -0.02em;
    color: #f8f6f4;
    margin: 0 20px 8px;
    display: flex;
    align-items: center;
    padding-left: 10px;
}

.dropdown__list-item:hover {
    color: #294b8e;
    background-color: #f8f6f4;
}

.dropdown input {
    padding: 14px 16px;
    border-radius: 8px;
    background-color: transparent;
    border: 1px solid #293b5f;
    color: #293b5f;
    font-size: 16px;
    margin-bottom: 8px;
}

.dropdown input::placeholder {
    color: #6f7e9e;
}

.btn-container {
    display: flex;
    justify-content: center;
    margin-top: 8px;
}

.btn-container button {
    border-radius: 8px;
    width: 156px;
    height: 48px;
    font-weight: 500;
    font-size: 16px;
    line-height: 112%;
    text-align: center;
}

.save-btn {
    background: #294b8e;
    color: #f8f6f4;
    margin-right: 8px;
}

.cancel-btn {
    background-color: transparent;
    color: #294b8e;
    border: 1px solid #294b8e;
}

.window-delete {
    left: 0;
    background-color: rgb(14 22 38 / 25%) !important;
}
</style>