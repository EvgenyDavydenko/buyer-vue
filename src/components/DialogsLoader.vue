<script>
import axios from "axios";

export default {
  props: {
    accessToken: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      dialogs: [],
    };
  },
  async mounted() {
    try {
      const response = await axios.post("http://localhost:80/graphql",
        {
          query: `
            query {
              getDialogsList {
                id user { phoneNumber }
              }
            }
          `,
        },
        {
          headers: {
            Authorization: `Bearer ${this.accessToken}`,
          },
        }
      );

      const dialogs = response.data.data.getDialogsList;
      console.log(dialogs);
      if (dialogs.length > 0) {
        this.dialogs = dialogs;
      } else {
        console.warn("Нет доступных диалогов.");
      }
    } catch (error) {
      console.error("Ошибка выполнения GraphQL-запроса:", error);
    }
  },
  render() {
    // Передаём список всех диалогов через слот
    return this.dialogs.length
      ? this.$slots.default({ dialogs: this.dialogs })
      : null;
  },
};
</script>