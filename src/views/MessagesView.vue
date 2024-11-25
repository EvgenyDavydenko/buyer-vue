<script>
import axios from "axios";
import RealtimeListener from "../components/RealtimeListener.vue";

export default {
  components: {
    RealtimeListener,
  },
  data() {
    return {
      messages: [],
      loading: false,
      error: null,
      newMessage: "",
    };
  },
  computed: {
    dialogId() {
      return this.$route.params.id; // Получаем dialogId из URL
    },
    accessToken() {
      return sessionStorage.getItem("accessToken");
    },
  },
  async mounted() {
    await this.fetchMessages();
  },
  methods: {
    async fetchMessages() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.post("http://localhost:80/graphql",
          {
            query: `
              query ($dialogId: ID!) {
                getDialogMessages(dialogId: $dialogId) {
                  id
                  sender { phoneNumber }
                  content
                  createdAt
                }
              }
            `,
            variables: {
              dialogId: this.dialogId,
            },
          },
          {
            headers: {
              Authorization: `Bearer ${this.accessToken}`,
            },
          }
        );

        if (response.data && response.data.data) {
          this.messages = response.data.data.getDialogMessages || [];
        } else {
          throw new Error("Неправильный формат ответа сервера.");
        }
      } catch (error) {
        console.error("Ошибка загрузки сообщений:", error);
        this.error = "Ошибка загрузки сообщений. Попробуйте позже.";
      } finally {
        this.loading = false;
      }
    },
    addNewMessage(message) {
      this.messages.push(message); // Добавляем новое сообщение в список
    },
        async sendMessage() {
      if (!this.newMessage.trim()) return;

      try {
        const response = await axios.post("http://localhost:80/graphql",
          {
            query: `
              mutation ($content: String!, $dialogId: ID!) {
                sendMessage(content: $content, dialogId: $dialogId) {
                  id
                  sender { phoneNumber }
                  content
                  createdAt
                }
              }
            `,
            variables: {
              content: this.newMessage,
              dialogId: this.dialogId,
            },
          },
          {
            headers: {
              Authorization: `Bearer ${this.accessToken}`,
            },
          }
        );

        if (response.data && response.data.data) {
          const sentMessage = response.data.data.sendMessage;
          this.addNewMessage(sentMessage); // Добавляем отправленное сообщение в список
          this.newMessage = ""; // Очищаем поле ввода
        } else {
          throw new Error("Ошибка отправки сообщения.");
        }
      } catch (error) {
        console.error("Ошибка при отправке сообщения:", error);
        this.error = "Не удалось отправить сообщение. Попробуйте позже.";
      }
    },
  },
};
</script>

<template>
  <div class="messages-view">
    <h1>Сообщения диалога</h1>
    <div v-if="loading">
      <p>Загрузка сообщений...</p>
    </div>
    <div v-else-if="error">
      <p>{{ error }}</p>
    </div>
    <div v-else>
      <ul>
        <li v-for="message in messages" :key="message.id">
          <strong>{{ message.sender.phoneNumber }}:</strong> {{ message.content }}
          <br />
          <small>{{ new Date(message.createdAt).toLocaleString() }}</small>
        </li>
      </ul>

      <!-- Поле ввода нового сообщения -->
      <div class="message-input">
        <textarea
          v-model="newMessage"
          placeholder="Введите сообщение..."
          rows="3"
        ></textarea>
        <button @click="sendMessage" :disabled="!newMessage.trim()">
          Отправить
        </button>
      </div>
    </div>
    <!-- Реальный тайм -->
    <RealtimeListener
      :token="accessToken"
      :dialogId="dialogId"
      @newMessage="addNewMessage"
    />
  </div>
</template>

<style scoped>
.messages-view ul {
  list-style: none;
  padding: 0;
}

.messages-view li {
  margin-bottom: 15px;
  border-bottom: 1px solid #ddd;
  padding-bottom: 10px;
}

.message-input {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
}

.message-input textarea {
  resize: none;
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  font-size: 14px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.message-input button {
  align-self: flex-end;
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.message-input button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>