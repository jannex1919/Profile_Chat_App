<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import NavUser from '@/components/NavUser.vue';
import { Sidebar, SidebarContent, SidebarHeader } from '@/components/ui/sidebar';

const friends = ref<any[]>([]);
const group = ref<any>(null);
const selectedFriendId = ref<number | null>(null);
const selectedGroup = ref<boolean>(false);


const emit = defineEmits<{
    chatSelected: [chat: any, type: 'friend' | 'group']
}>();

const fetchFriends = () => {
    console.log('Fetching friends...');
    axios.get('/friends/list', {
        headers: {
            'Accept': 'application/json',
            'X-Requested-With': 'XMLHttpRequest'
        }
    })
        .then((response) => {
            console.log("Friends API Response:", response);

            let friendsArray: any[] = [];

            if (Array.isArray(response.data)) {
                friendsArray = response.data;
                console.log("Data is direct array");
            } else if (response.data.data && Array.isArray(response.data.data)) {
                friendsArray = response.data.data;
                console.log("Data is in response.data.data");
            } else if (response.data.friends && Array.isArray(response.data.friends)) {
                friendsArray = response.data.friends;
                console.log("Data is in response.data.friends");
            } else if (response.data.props && response.data.props.friends && Array.isArray(response.data.props.friends)) {
                friendsArray = response.data.props.friends;
                console.log("Data is in response.data.props.friends (Inertia structure)");
            } else {
                console.log("No matching array structure found");
                console.log("Available keys in response.data:", Object.keys(response.data || {}));
            }

            friends.value = friendsArray.map((friend: any) => ({
                id: friend.id,
                name: friend.name,
                email: friend.email,
                avatar: friend.avatar,
                status: friend.status,
                last_seen: friend.last_seen,
                last_message: friend.last_message,
                last_message_time: friend.last_message_time,
                href: friend.href || `/chat/${friend.id}`,
                avatarError: false // Track avatar loading errors
            }));

            console.log("Mapped friends:", friends.value);
        })
        .catch((error) => {
            console.error('Error fetching friends:', error);
        });
};

const fetchGroup = () => {
    console.log('Fetching group...');
    axios.get('/groups/list', {
        headers: {
            'Accept': 'application/json',
            'X-Requested-With': 'XMLHttpRequest'
        }
    })
        .then((response) => {
            console.log("Group API Response:", response);

            if (response.data.data && response.data.data.length > 0) {
                group.value = response.data.data[0];
                console.log("Group loaded:", group.value);
            }
        })
        .catch((error) => {
            console.error('Error fetching group:', error);
        });
};

const selectFriend = (friend: any) => {
    selectedFriendId.value = friend.id;
    selectedGroup.value = false;
    emit('chatSelected', friend, 'friend');
    console.log('Selected friend:', friend);
};

const selectGroupChat = () => {
    selectedFriendId.value = null;
    selectedGroup.value = true;
    emit('chatSelected', group.value, 'group');
    console.log('Selected group chat:', group.value);
};

// Handle image loading errors
const handleImageError = (friend: any) => {
    console.error('Failed to load avatar for:', friend.name, 'URL:', friend.avatar);
    friend.avatarError = true;
};

onMounted(() => {
    fetchFriends();
    fetchGroup();
});

function statusColor(status: string) {
    switch (status) {
        case 'online': return 'bg-emerald-400';
        case 'offline': return 'bg-gray-400';
        case 'away': return 'bg-amber-400';
        default: return 'bg-gray-400';
    }
}

function formatTime(timestamp: string) {
    if (!timestamp) return '';
    const date = new Date(timestamp);
    const now = new Date();
    const diff = now.getTime() - date.getTime();
    const hours = Math.floor(diff / (1000 * 60 * 60));

    if (hours < 1) return 'now';
    if (hours < 24) return `${hours}h`;
    const days = Math.floor(hours / 24);
    if (days < 7) return `${days}d`;
    return date.toLocaleDateString();
}

function truncateMessage(message: string, length: number = 30) {
    if (!message) return '';
    return message.length > length ? message.substring(0, length) + '...' : message;
}

// Check if avatar URL is valid
function hasValidAvatar(friend: any) {
    return friend.avatar && !friend.avatarError && friend.avatar.trim() !== '';
}
</script>

<template>
    <Sidebar collapsible="icon" variant="inset" class="bg-slate-900 border-r border-slate-700/50">
        <!-- Header Section -->
        <SidebarHeader class="bg-gradient-to-r from-slate-800 to-slate-900 border-b border-slate-700/50 px-4 py-6">
            <div class="space-y-4">
                <!-- Logo/Title Area -->
                <div class="flex items-center space-x-3">
                    <div class="w-8 h-8 bg-gradient-to-br from-teal-500 via-teal-600 to-emerald-600 rounded-lg flex items-center justify-center">
                        🗣️
                    </div>
                    <h1 class="text-xl font-bold text-white">Messages</h1>
                </div>

                <!-- User Profile -->
                <div class="bg-slate-800/50 rounded-xl p-3 border border-slate-700/30">
                    <NavUser />
                </div>
            </div>
        </SidebarHeader>

        <!-- Chat List Content -->
        <SidebarContent class="bg-slate-900 overflow-y-auto scrollbar-thin scrollbar-thumb-slate-700 scrollbar-track-slate-900">
            <!-- Stats Section -->
            <div class="px-4 py-4">
                <div class="bg-slate-800/50 rounded-lg p-3 border border-slate-700/30">
                    <div class="flex items-center justify-between text-sm">
                        <div class="flex items-center space-x-2">
                            <div class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></div>
                            <span class="text-slate-300 font-medium">{{ friends.length }} Active</span>
                        </div>
                        <div v-if="group" class="flex items-center space-x-1 text-slate-400">
                            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                                <path d="M13 6a3 3 0 11-6 0 3 3 0 016 0zM18 8a2 2 0 11-4 0 2 2 0 014 0zM14 15a4 4 0 00-8 0v3h8v-3z"/>
                            </svg>
                            <span class="text-sm font-medium">{{ group.member_count }}</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Group Chat Section -->
            <div v-if="group" class="px-4 pb-2">
                <div
                    @click="selectGroupChat"
                    :class="[
                        'group relative flex items-center gap-3 rounded-xl px-3 py-3.5 cursor-pointer transition-all duration-200 border',
                        selectedGroup
                            ? 'bg-gradient-to-r from-indigo-600/20 to-purple-600/20 border-indigo-500/30 shadow-lg shadow-indigo-500/10'
                            : 'hover:bg-slate-800/60 border-slate-700/30 hover:border-slate-600/50'
                    ]"
                >
                    <!-- Group Avatar -->
                    <div class="relative">
                        <div class="absolute -bottom-1 -right-1 bg-emerald-400 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center font-bold border-2 border-slate-900 shadow-sm">
                            {{ group.member_count }}
                        </div>
                    </div>

                    <!-- Group Info -->
                    <div class="flex-1 min-w-0">
                        <div class="flex items-center justify-between mb-1">
                            <span class="text-sm font-semibold text-white group-hover:text-indigo-300 transition-colors">
                                {{ group.name }}
                            </span>
                            <span v-if="group.last_message_time" class="text-xs text-slate-400">
                                {{ formatTime(group.last_message_time) }}
                            </span>
                        </div>
                        <p class="text-xs text-slate-400 truncate">
                            {{ group.description }}
                        </p>
                    </div>

                    <div v-if="selectedGroup" class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-8 bg-gradient-to-b from-indigo-500 to-purple-600 rounded-r-full"></div>
                </div>
            </div>

            <!-- Direct Messages Section -->
            <div class="px-4 py-2">
                <div class="flex items-center justify-between mb-3">
                    <h3 class="text-xs font-semibold text-slate-400 uppercase tracking-wider px-2">Messages</h3>
                    <span class="text-xs text-slate-500 bg-slate-800/50 px-2 py-1 rounded-full">{{ friends.length }}</span>
                </div>

                <div class="space-y-1">
                    <div v-for="friend in friends" :key="friend.id">
                        <div
                            @click="selectFriend(friend)"
                            :class="[
                                'group relative flex items-center gap-3 rounded-xl px-3 py-3 cursor-pointer transition-all duration-200 border',
                                selectedFriendId === friend.id
                                    ? 'bg-gradient-to-r from-emerald-600/20 to-teal-600/20 border-emerald-500/30 shadow-lg shadow-emerald-500/10'
                                    : 'hover:bg-slate-800/60 border-slate-700/30 hover:border-slate-600/50'
                            ]"
                        >
                            <!-- Avatar -->
                            <div class="relative">
                                <!-- Image Avatar -->
                                <div
                                    v-if="hasValidAvatar(friend)"
                                    class="h-10 w-10 rounded-xl overflow-hidden ring-2 ring-slate-700/50 group-hover:ring-slate-600/50 transition-all"
                                >
                                    <img
                                        :src="friend.avatar"
                                        :alt="friend.name"
                                        class="h-full w-full object-cover"
                                        @error="handleImageError(friend)"
                                    />
                                </div>
                                <!-- Fallback Avatar -->
                                <div
                                    v-else
                                    class="h-10 w-10 rounded-xl bg-gradient-to-br from-slate-600 to-slate-700 flex items-center justify-center text-sm font-bold text-white ring-2 ring-slate-700/50 group-hover:ring-slate-600/50 transition-all shadow-lg"
                                >
                                    {{ friend.name.charAt(0).toUpperCase() }}
                                </div>
                                <!-- Status dot -->
                                <span
                                    class="absolute -bottom-0.5 -right-0.5 h-3.5 w-3.5 rounded-full border-2 border-slate-900 shadow-sm"
                                    :class="statusColor(friend.status)"
                                />
                            </div>

                            <!-- Friend Info -->
                            <div class="flex-1 min-w-0">
                                <div class="flex items-center justify-between mb-1">
                                    <span class="text-sm font-semibold truncate text-white group-hover:text-emerald-300 transition-colors">
                                        {{ friend.name }}
                                    </span>
                                    <span v-if="friend.last_message_time" class="text-xs text-slate-400 flex-shrink-0">
                                        {{ formatTime(friend.last_message_time) }}
                                    </span>
                                </div>
                                <div class="flex items-center justify-between">
                                    <span class="text-xs text-slate-400 truncate">
                                        <span v-if="friend.last_message">
                                            {{ truncateMessage(friend.last_message) }}
                                        </span>
                                        <span v-else class="capitalize">
                                            {{ friend.status }}
                                            <span v-if="friend.last_seen" class="text-slate-500"> • {{ friend.last_seen }}</span>
                                        </span>
                                    </span>
                                </div>
                            </div>

                            <!-- Active indicator -->
                            <div v-if="selectedFriendId === friend.id" class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-8 bg-gradient-to-b from-emerald-500 to-teal-600 rounded-r-full"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Empty States -->
            <div v-if="!group && friends.length === 0" class="p-6 text-center text-slate-400">
                <div class="w-16 h-16 mx-auto mb-4 bg-slate-800/50 rounded-full flex items-center justify-center">
                    <svg class="w-8 h-8 text-slate-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M18 10c0 3.866-3.582 7-8 7a8.841 8.841 0 01-4.083-.98L2 17l1.338-3.123C2.493 12.767 2 11.434 2 10c0-3.866 3.582-7 8-7s8 3.134 8 7zM7 9H5v2h2V9zm8 0h-2v2h2V9zM9 9h2v2H9V9z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <p class="text-sm font-medium mb-1 text-slate-300">No conversations yet</p>
                <p class="text-xs text-slate-500">Start by adding some friends</p>
            </div>

            <div v-if="friends.length === 0 && group" class="p-4 text-center text-slate-400 text-xs">
                <p>No direct messages</p>
            </div>
        </SidebarContent>

        <!-- Footer -->

    </Sidebar>

    <slot />
</template>

<style scoped>

.scrollbar-thin {
    scrollbar-width: thin;
}

.scrollbar-thumb-slate-700 {
    --scrollbar-thumb: #374151;
}

.scrollbar-track-slate-900 {
    --scrollbar-track: #0f172a;
}
</style>
