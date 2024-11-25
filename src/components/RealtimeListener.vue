<script>
import Echo from "laravel-echo";
import Pusher from "pusher-js";

export default {
  props: {
    token: {
      type: String,
      required: true,
    },
    dialogId: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      messages: [],
    };
  },
  mounted() {
    window.Pusher = Pusher;
    window.Pusher.logToConsole = true;

    console.log("mounted ...");

    // Создание экземпляра Laravel Echo
    this.echo = new Echo({
      broadcaster: "reverb",
      key: "d6afo3fjzrmflyhq6mrg",
      wsHost: "localhost",
      wsPort: 80,
      wssPort: 443,
      forceTLS: false,
      encrypted: true,
      authEndpoint: "http://localhost/broadcasting/auth",
      auth: {
        headers: {
          Authorization: `Bearer ${this.token}`,
          Accept: "application/json",
        },
      },
      withCredentials: true,
      enabledTransports: ["ws"],
    });

    this.echo.connector.pusher.connection.bind("error", (err) => {
        console.error("Pusher connection error:", err);
    });
    this.echo.connector.pusher.connection.bind("state_change", (state) => {
        console.log("Connection state changed:", state);
    });

    console.log("Subscribing to channel...");
    // Подписываемся на приватный канал
    this.echo.private(`dialog.${this.dialogId}`).listen(".messageSent", (event) => {
            console.log("Message received:", event);
            this.messages.push(event); // Сохраняем полученное событие
        })
        .error((error) => {
            console.error("Subscription error:", error);
        });
  },
  beforeDestroy() {
    // Очищаем соединение при уничтожении компонента
    if (this.echo) {
      this.echo.disconnect();
    }
  },
};
</script>

<template>
  <div>
    <p>WebSocket Connection Established</p>
    <div v-if="messages.length">
      <h3>Received Messages:</h3>
      <ul>
        <li v-for="(message, index) in messages" :key="index">
          {{ message }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 5px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}
</style>