<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import AppSidebar from '@/components/AppSidebar.vue';
import { Head, usePage } from '@inertiajs/vue3';
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
import axios from 'axios';

const chatmate = ref<any | null>(null);
const chatId = ref<number | null>(null);
const messages = ref<any[]>([]);
const newMessage = ref('');
const messagesContainer = ref<HTMLElement | null>(null);
const chatType = ref<'friend' | 'group' | null>(null);

// Polling interval reference
let pollingInterval: NodeJS.Timeout | null = null;

// current logged-in user
const currentUser = usePage().props.auth.user;

async function handleChatSelected(chat: any, type: 'friend' | 'group') {
    chatmate.value = chat;
    chatType.value = type;

    // Clear existing interval when switching chats
    stopPolling();

    try {
        if (type === 'friend') {
            // Handle friend chat
            const res = await axios.get(`/chat/${chat.id}`);
            chatId.value = res.data.id;
            messages.value = res.data.messages;
        } else if (type === 'group') {
            // Handle group chat
            const res = await axios.get('/group-chat/messages');
            chatId.value = 1;
            messages.value = res.data.messages || [];
        }

        // Scroll to bottom after loading messages
        await nextTick();
        scrollToBottom();

        // Start polling for new messages
        startPolling();
    } catch (err) {
        console.error("Error loading chat:", err);
    }
}

async function sendMessage() {
    if (!newMessage.value.trim()) return;

    try {
        let res;

        if (chatType.value === 'friend' && chatId.value) {
            // Send direct message to friend
            res = await axios.post(`/chat/${chatId.value}/send`, {
                message: newMessage.value,
            });
        } else if (chatType.value === 'group') {
            // Send message to group
            res = await axios.post('/group-chat/send', {
                message: newMessage.value,
            });
        } else {
            return;
        }

        // Push the new message
        messages.value.push(res.data.message || res.data);
        newMessage.value = '';

        // Scroll to bottom after sending message
        await nextTick();
        scrollToBottom();
    } catch (err) {
        console.error("Error sending message:", err);
    }
}

async function fetchMessages() {
    if (!chatmate.value || !chatType.value) return;

    try {
        let res;

        if (chatType.value === 'friend') {
            res = await axios.get(`/chat/${chatmate.value.id}`);
        } else if (chatType.value === 'group') {
            res = await axios.get('/group-chat/messages');
        } else {
            return;
        }

        const newMessages = chatType.value === 'friend' ? res.data.messages : res.data.messages || [];

        // Only update if there are new messages to prevent unnecessary re-renders
        if (newMessages.length !== messages.value.length) {
            const wasAtBottom = isScrolledToBottom();
            messages.value = newMessages;

            // Auto-scroll only if user was already at bottom
            if (wasAtBottom) {
                await nextTick();
                scrollToBottom();
            }
        }
    } catch (err) {
        console.error("Error fetching messages:", err);
    }
}

function startPolling() {
    if (pollingInterval) return; // Prevent multiple intervals

    pollingInterval = setInterval(() => {
        fetchMessages();
    }, 1500);
}

function stopPolling() {
    if (pollingInterval) {
        clearInterval(pollingInterval);
        pollingInterval = null;
    }
}

function scrollToBottom() {
    if (messagesContainer.value) {
        messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
}

function isScrolledToBottom() {
    if (!messagesContainer.value) return true;

    const { scrollTop, scrollHeight, clientHeight } = messagesContainer.value;
    return scrollTop + clientHeight >= scrollHeight - 10; // 10px threshold
}

// Get display name based on chat type
function getDisplayName() {
    if (!chatmate.value) return '';
    return chatType.value === 'group' ? chatmate.value.name : chatmate.value.name;
}

// Get status display
function getStatusDisplay() {
    if (!chatmate.value) return '';
    if (chatType.value === 'group') {
        return `${chatmate.value.member_count} members`;
    }
    return chatmate.value.status || 'offline';
}

// Get avatar/icon
function getAvatarDisplay() {
    if (!chatmate.value) return '';

    if (chatType.value === 'group') {
        return '👥'; // Group emoji
    }
    return chatmate.value.name.charAt(0);
}

// Get avatar background class
function getAvatarBgClass() {
    if (chatType.value === 'group') {
        return 'bg-gradient-to-br from-violet-500 to-purple-600';
    }
    return 'bg-gradient-to-br from-blue-500 to-indigo-600';
}

// Get status indicator class
function getStatusClass() {
    if (chatType.value === 'group') {
        return 'bg-emerald-500'; // Always online for group
    }
    return chatmate.value?.status === 'online' ? 'bg-emerald-500' : 'bg-slate-400';
}

// Format message display for group messages
function formatMessageDisplay(msg: any) {
    if (chatType.value === 'group' && msg.user_id !== currentUser.id) {
        return `${msg.user_name}: ${msg.message}`;
    }
    return msg.message;
}

// Lifecycle hooks
onMounted(() => {
    // Start polling if there's already an active chat
    if (chatId.value) {
        startPolling();
    }
});

onUnmounted(() => {
    // Clean up interval when component is destroyed
    stopPolling();
});
</script>

<template>
    <Head title="Chat" />

    <AppLayout class="min-h-screen bg-gradient-to-br from-slate-50 via-white to-gray-100 dark:from-slate-950 dark:via-slate-900 dark:to-gray-900">
        <AppSidebar @chatSelected="handleChatSelected" />
        <div class="flex h-screen bg-gray-100 dark:bg-gray-900">
            <div class="flex-1 flex flex-col bg-white dark:bg-gray-800">


            <div class="flex-1 flex flex-col bg-white">

                <template v-if="chatmate">
                    <div class="flex items-center justify-between border-b border-gray-200 dark:border-gray-700 px-6 py-4 bg-white dark:bg-gray-800 shadow-sm">
                        <div class="flex items-center gap-4">
                            <div class="relative">
                                <div
                                    class="h-12 w-12 rounded-full flex items-center justify-center text-white font-bold text-lg shadow-lg"
                                    :class="getAvatarBgClass()">
                                    {{ getAvatarDisplay() }}
                                </div>
                                <span
                                    class="absolute -bottom-1 -right-1 h-4 w-4 rounded-full border-3 border-white shadow-sm"
                                    :class="getStatusClass()" />
                            </div>
                            <div>
                                <h2 class="font-semibold text-gray-900 dark:text-gray-100 text-lg">{{ getDisplayName() }}</h2>
                                <p class="text-sm text-gray-500 dark:text-gray-400 capitalize">{{ getStatusDisplay() }}</p>
                            </div>
                        </div>

                    </div>

                    <!-- Messages Container -->
                    <div
                        ref="messagesContainer"
                        class="flex-1 overflow-y-auto p-6 bg-gray-50/30">

                        <!-- Messages -->
                        <div class="space-y-4">
                            <div v-for="msg in messages" :key="msg.id" :class="[
                                'flex items-end gap-3',
                                msg.user_id === currentUser.id ? 'justify-end' : 'justify-start'
                            ]">
                                <!-- Avatar for other users -->
                                <div v-if="msg.user_id !== currentUser.id" class="w-8 h-8 rounded-full bg-gradient-to-br from-blue-500 to-indigo-600 flex items-center justify-center text-white text-sm font-semibold shadow-sm">
                                    {{ msg.user_name ? msg.user_name.charAt(0).toUpperCase() : 'U' }}
                                </div>

                                <!-- Message Bubble -->
                                <div :class="[
                                    'max-w-md px-4 py-3 rounded-2xl text-sm shadow-sm',
                                    msg.user_id === currentUser.id
                                        ? 'bg-gradient-to-r from-blue-500 to-indigo-600 text-white rounded-br-md'
                                        : 'bg-white text-gray-800 border border-gray-100 rounded-bl-md'
                                ]">
                                    <p class="leading-relaxed">{{ formatMessageDisplay(msg) }}</p>
                                    <div :class="[
                                        'flex items-center gap-2 mt-2 text-xs opacity-70',
                                        msg.user_id === currentUser.id ? 'text-blue-100' : 'text-gray-500'
                                    ]">
                                        <span>{{ new Date(msg.created_at || Date.now()).toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' }) }}</span>
                                        <svg v-if="msg.user_id === currentUser.id" class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                                            <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"/>
                                        </svg>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Empty state for messages -->
                        <div v-if="messages.length === 0" class="text-center py-16">
                            <div class="w-20 h-20 mx-auto mb-6 bg-gradient-to-br from-blue-100 to-indigo-100 rounded-full flex items-center justify-center">
                                <div class="text-3xl">
                                    {{ chatType === 'group' ? '👥' : '💬' }}
                                </div>
                            </div>
                            <h3 class="text-lg font-semibold text-gray-800 mb-2">
                                {{ chatType === 'group' ? 'Welcome to the group!' : `Start chatting with ${chatmate.name}` }}
                            </h3>
                            <p class="text-gray-500 text-sm">
                                {{ chatType === 'group' ? 'No messages in this group yet. Be the first to say hello!' : 'Send a message to begin your conversation.' }}
                            </p>
                        </div>
                    </div>

                    <!-- Message Input -->
                    <div class="border-t border-gray-100 p-6 bg-white">
                        <div class="flex items-end gap-4">
                            <!-- Input Container -->
                            <div class="flex-1 relative">
                                <textarea
                                    v-model="newMessage"
                                    @keyup.enter.exact="sendMessage"
                                    @keydown.enter.exact.prevent
                                    rows="1"
                                    :placeholder="chatType === 'group' ? 'Message everyone...' : 'Type your message...'"
                                    class="w-full resize-none rounded-2xl border border-gray-200 px-4 py-3 pr-12 text-sm text-black focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent placeholder-gray-400 bg-gray-50 hover:bg-white transition-colors"
                                    style="min-height: 48px; max-height: 120px;"
                                />
                            </div>

                            <!-- Send Button -->
                            <button
                                @click="sendMessage"
                                :disabled="!newMessage.trim()"
                                :class="[
                                    'p-3 rounded-full transition-all transform hover:scale-105 active:scale-95 shadow-lg',
                                    newMessage.trim()
                                        ? (chatType === 'group'
                                            ? 'bg-gradient-to-r from-emerald-500 to-green-600 text-white hover:shadow-emerald-200'
                                            : 'bg-gradient-to-r from-blue-500 to-indigo-600 text-white hover:shadow-blue-200')
                                        : 'bg-gradient-to-br from-teal-500 via-teal-600 to-emerald-600 cursor-not-allowed hover:scale-100'
                                ]">
                              📩
                            </button>
                        </div>
                    </div>
                </template>

                <!-- If no chatmate selected -->
                <div v-else class="flex-1 flex items-center justify-center bg-gradient-to-br from-blue-50 via-white to-indigo-50">
                    <div class="text-center max-w-md mx-auto px-8">
                        <div class="w-24 h-24 mx-auto mb-8 bg-gradient-to-br from-teal-500 via-teal-600 to-emerald-600 rounded-full flex items-center justify-center shadow-2xl">
                            <div class="text-4xl text-white">🗣️</div>
                        </div>
                        <h2 class="text-2xl font-bold text-gray-800 mb-4">Welcome to Your Chat</h2>
                        <p class="text-gray-600 mb-8 leading-relaxed">
                            Connect with your friends and groups. Select a conversation from the sidebar to start chatting and sharing moments together.
                        </p>
                    </div>
                </div>
            </div>
        </div>
        </div>
    </AppLayout>
</template>

<style scoped>


/* Smooth animations */
.message-enter-active,
.message-leave-active {
    transition: all 0.3s ease;
}


</style>
