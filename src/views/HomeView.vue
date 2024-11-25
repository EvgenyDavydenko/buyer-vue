<script>
import DialogsLoader from "../components/DialogsLoader.vue";

export default {
  components: {
    DialogsLoader,
  },

  data() {
    return {
      accessToken: null,
      user: null,
    };
  },

  methods: {
    openDialog(dialogId) {
      this.$router.push(`/dialog/${dialogId}`); // Переход на страницу сообщений
    },
  },

  mounted() {
    // Получаем токен и данные пользователя из sessionStorage
    this.accessToken = sessionStorage.getItem("accessToken");
    const user = sessionStorage.getItem("user");

    // Проверяем валидность данных пользователя
    this.user = user ? JSON.parse(user) : null;

    // Если токена нет, можно перенаправить на страницу входа
    if (!this.accessToken) {
      this.$router.push("/login");
    }
  }
};
</script>

<template>
  <div class="home">
    <div v-if="user">
      <h1>Привет, {{ user.lastName }} {{ user.firstName }}!</h1>
      <p>Вы успешно вошли в систему.</p>

      <DialogsLoader :accessToken="accessToken">
        <template v-slot:default="{ dialogs }">
          <div v-if="dialogs.length > 0">
            <h2>Ваши диалоги:</h2>
            <ul>
              <li
                v-for="dialog in dialogs"
                :key="dialog.id"
                @click="openDialog(dialog.id)"
                class="dialog-item"
              >
                {{ dialog.user.phoneNumber }} (ID: {{ dialog.id }})
              </li>
            </ul>
          </div>
          <div v-else>
            <p>Диалоги отсутствуют.</p>
          </div>
        </template>
      </DialogsLoader>
    </div>

    <div v-else>
      <h1>Добро пожаловать!</h1>
      <p>Пожалуйста, <a href="/login">войдите в систему</a>, чтобы продолжить.</p>
    </div>
  </div>
</template>

<style>
  .home {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  h1 {
    margin-bottom: 10px;
  }

  p {
    font-size: 1.2rem;
  }
</style>
