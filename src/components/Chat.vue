<template>
  <div class="h-100 bg-secondary d-flex">
    <div class="h-100 d-flex flex-column align-items-stretch flex-shrink-0 bg-white" style="width: 380px">
      <a href="#" @click.prevent class="d-flex align-items-center flex-shrink-0 p-3 link-dark text-decoration-none border-bottom">
        <span class="fs-5 fw-semibold">Conversations</span>
      </a>
      <div class="list-group list-group-flush border-bottom overflow-auto">
        <User v-for="user in users" :key="user.userID" :user="user" :selected="selectedUser === user" @select="onSelectUser(user)" />
      </div>
    </div>
    <MessagePanel v-if="selectedUser" :user="selectedUser" @message="onMessage" />
  </div>
</template>

<script>
import User from '@/components/User'
import MessagePanel from '@/components/MessagePanel'
import socket from '@/socket'

export default {
    name: 'Chat',
    components: {
        User,
        MessagePanel
    },
    data: () => ({
        selectedUser: null,
        users: []
    }),
    methods: {
        onSelectUser(user) {
            this.selectedUser = user
            user.hasNewMessages = false
        },
        onMessage(content) {
            if (this.selectedUser) {
                socket.emit('private message', {
                    content,
                    to: this.selectedUser.userID
                })
                this.selectedUser.messages.push({
                    content,
                    fromSelf: true
                })
            }
        }
    },
    created() {
        socket.on('connect', () => {
            this.users.forEach((user) => {
                if (user.self) {
                    user.connected = true
                }
            })
        }),

        socket.on('disconnect', () => {
            this.users.forEach((user) => {
                if (user.self) {
                    user.connected = false
                }
            })
        })

        const initReactiveProperties = (user) => {
            user.hasNewMessages = false
        }

        socket.on('users', (users) => {
            users.forEach((user) => {
                user.messages.forEach((message) => {
                    message.fromSelf = message.from === socket.userID
                })
                for (let i = 0; i < this.users.length; i++) {
                    const existingUser = this.users[i]
                    if (existingUser.userID === user.userID) {
                        existingUser.connected = user.connected
                        existingUser.messages = user.messages
                        return
                    }
                }
                user.self = user.userID === socket.userID
                initReactiveProperties(user)
                this.users.push(user)
            })
            this.users.sort((a, b) => {
                if (a.self) return -1
                if (b.self) return 1
                if (a.username < b.username) return -1
                return a.username > b.username ? 1 : 0
            })
        })

        socket.on('user connected', (user) => {
            for (let i = 0; i < this.users.length; i++) {
                const existingUser = this.users[i]
                if (existingUser.userID === user.userID) {
                    existingUser.connected = true
                    return
                }
            }
            initReactiveProperties(user)
            this.users.push(user)
        })

        socket.on('user disconnected', (id) => {
            const idx = this.users.findIndex(u => u.userID === id)
            this.users[idx].connected = false
        })

        socket.on('private message', ({content, from, to}) => {
            for (let i = 0; i < this.users.length; i++) {
                const user = this.users[i]
                const fromSelf = socket.userID === from
                if (user.userID === (fromSelf ? to : from)) {
                    user.messages.push({
                        content,
                        fromSelf
                    })
                    if (user !== this.selectedUser) {
                        user.hasNewMessages = true
                    }
                    break
                }
            }
        })
    },

    unmounted() {
        socket.off('connect')
        socket.off('disconnect')
        socket.off('users')
        socket.off('user connected')
        socket.off('user disconnected')
        socket.off('private message')
    }

}
</script>