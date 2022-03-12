<template>
  <div class="d-flex justify-content-center mt-5">
    <form @submit.prevent="onSubmit">
      <div class="mb-3">
        <label for="username" class="form-label">Enter your username</label>
        <input
          type="text"
          class="form-control"
          id="username"
          v-model="username"
          placeholder="Your username..."
        />
      </div>
      <button type="submit" class="btn btn-primary" :disabled="!isValid">Send</button>
    </form>
  </div>
</template>

<script>
export default {
  name: "SelectUsername",
  data: () => ({
    username: "",
  }),
  computed: {
    isValid() {
      return this.username.length > 2;
    },
  },
  methods: {
    onSubmit() {
      this.$emit("selected", this.username);
    },
  },
};
</script>