<template>
    <div class="pt-6">
        <div class="max-w-7xl mx-auto">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">You're chats!</div>
            </div>
        </div>
    </div>
    <div class="flex items-start border-2 border-sky-gray mt-5 max-w-7xl mx-auto sm:px-6 lg:px-8 bg-white shadow-sm sm:rounded-lg">
        <div class="w-1/2 p-4 mr-4 bg-white border-solid">
            <h3 class="text-gray-700 mb-4 text-lg">Chats</h3>
            <div v-if="chats">
                <div v-for="chat in chats" class="pb-4 mb-4">
                    <Link :href="route('chats.show', chat.id)">
                        <div>
                            <div class="border-2 border-sky-gray p-4 rounded-lg">
                                <div class="flex">
                                    <p>{{ chat.title }}</p>
                                </div>
                                <div :class="['p-4 flex justify-between items-center',
                                    chat.unreadable_count !== 0 ? 'bg-sky-50' : ''
                                ]">
                                    <div class="text-sm">
                                        <p class="text-gray-600 text-lg mb-3">{{ chat.last_message.user_name }}</p>
                                        <p class="mb-2 text-gray-500">{{ chat.last_message.body }}</p>
                                        <p class="italic text-gray-400">{{ chat.last_message.time }}</p>
                                    </div>

                                    <div v-if="chat.unreadable_count !== 0">
                                        <p class="text-xs rounded-full bg-sky-500 text-white px-2 py-1">{{
                                                chat.unreadable_count
                                            }}</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </Link>
                </div>
            </div>

        </div>
        <div class="w-1/2 p-4 sm:px-6 lg:px-8 bg-white overflow-hidden shadow-sm sm:rounded-lg">
            <div class="flex items-center  mb-4 justify-between">
                <h3 class="text-gray-700 text-lg">Users</h3>
                <div v-if="isGroup" class="flex items-center">
                    <input class="h-8 mr-4 border border-gray-300 rounded-full" type="text" placeholder="group title"
                           v-model="title">
                    <a @click.prevent="storeGroup"
                       :class="['inline-block mr-2  text-white text-xs px-3 py-2 rounded-lg',
                       this.userIds.length > 1 ? 'bg-green-600' : 'bg-green-300']" href="#">Go chat</a>
                    <a @click.prevent="refreshUserIds"
                       class="inline-block bg-indigo-600 text-white text-xs px-3 py-2 rounded-lg" href="#">X</a>
                </div>
            </div>
            <div v-if="users">
                <div v-for="user in users" class="flex justify-between items-center pb-4 mb-4 border-2 border-sky-gray p-4 rounded-lg">
                    <div class="flex items-center">
                        <p class="mr-2">{{ user.id }}</p>
                        <p class="mr-4">{{ user.name }}</p>
                        <a @click.prevent="store(user.id)"
                           class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full sm:w-auto px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800" href="#">Message</a>
                    </div>
                    <div v-if="isGroup">
                        <input @click="toggleUsers(user.id)" type="checkbox">
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import {Link} from '@inertiajs/vue3';
import Main from "@/Layouts/Main.vue";

export default {
    name: "Index",

    props: [
        'users',
        'chats'
    ],

    data() {
        return {
            isGroup: false,
            userIds: [],
            title: null
        }
    },

    created() {
        window.Echo.private(`users.${this.$page.props.auth.user.id}`)
            .listen('.store-message-status', res => {
                this.chats.filter(chat => {
                    if (chat.id === res.chat_id) {
                        chat.unreadable_count = res.count
                        chat.last_message = res.message
                    }
                })

            })
    },

    components: {
        Link
    },

    layout: Main,

    methods: {
        store(id) {
            this.$inertia.post('/chats', {title: null, users: [id]})
        },

        storeGroup() {
            if (this.userIds.length < 2) return
            this.$inertia.post('/chats', {title: this.title, users: this.userIds})
        },

        toggleUsers(id) {
            let index = this.userIds.indexOf(id)
            if (index === -1) {
                this.userIds.push(id)
            } else {
                this.userIds.splice(index, 1)
            }
        },

        refreshUserIds() {
            this.userIds = []
            this.isGroup = false
        }
    }

}
</script>

<style scoped>

</style>
