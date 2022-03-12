<template>
  <div class="p-3 w-100 d-flex flex-column justify-content-between">
    <ul class="pe-3 messages flex-grow-1 overflow-auto mb-3 d-flex flex-column">
      <li v-for="(message, index) in user.messages" :key="index" class="rounded p-3 mb-3" style="width: fit-content" :class="{'message-from-self': message.fromSelf, 'message-from-someone': !message.fromSelf}">
        {{ message.content }}
      </li>
    </ul>

    <form @submit.prevent="onSubmit" class="w-100">
      <div class="input-group">
        <textarea
          class="form-control custom-control"
          rows="3"
          style="resize: none"
          v-model="input"
        ></textarea>
        <button class="input-group-addon btn btn-primary d-flex align-items-center" :disabled="!isValid">Send</button>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  name: "MessagePanel",
  emits: ["message"],
  props: {
    user: Object,
  },
  data: () => ({
    input: "",
  }),
  methods: {
    onSubmit() {
      this.$emit("message", this.input);
      this.input = "";
    },
  },
  computed: {
    isValid() {
      return this.input.length > 0;
    },
  },
};
</script>

<style>
ul.messages {
  padding: 0;
  margin: 0;
  list-style-type: none;
}

ul.messages .message-from-self {
    align-self: flex-end;
    background:#cff4fc !important;
}

ul.messages .message-from-someone {
    align-self: flex-start;
    background: #d2f4ea !important;
}
</style>