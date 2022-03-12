<template>
  <a
    href="#"
    class="list-group-item list-group-item-action py-3 lh-tight" :class="{active: selected}"
    @click="onClick"
  >
    <div class="d-flex w-100 align-items-center justify-content-between">
      <strong class="d-flex align-items-center">{{user.username}} {{user.self ? ' (yourself)' : ''}} <span class="badge rounded-pill ms-1" :class="{'bg-danger': status === 'offline', 'bg-success': status === 'online'}">{{status}}</span></strong>
      <i class="bi bi-envelope-exclamation text-danger" v-if="user.hasNewMessages"></i>
    </div>
    <div class="col-10 mb-1 small">
      {{lastMessage}}
    </div>
  </a>
</template>

<script>
export default {
    name: 'User',
    props: {
        user: Object,
        selected: Boolean
    },
    methods: {
        onClick() {
            this.$emit('select')
        }
    },
    computed: {
        status() {
            return this.user.connected ? 'online' : 'offline'
        },
        lastMessage() {
            return this.user.messages[this.user.messages.length - 1] ? this.user.messages[this.user.messages.length - 1].content : ''
        }
    }
}
</script>
