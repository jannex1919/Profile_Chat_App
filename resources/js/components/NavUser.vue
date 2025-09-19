<script setup lang="ts">
import UserInfo from '@/components/UserInfo.vue';
import { DropdownMenu, DropdownMenuContent, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { SidebarMenu, SidebarMenuButton, SidebarMenuItem, useSidebar } from '@/components/ui/sidebar';
import { usePage } from '@inertiajs/vue3';
import { ChevronsUpDown, X } from 'lucide-vue-next';
import { ref } from 'vue';
import UserMenuContent from './UserMenuContent.vue';

const page = usePage();
const user = page.props.auth.user;
const { isMobile, state } = useSidebar();
const isDropdownOpen = ref(false);

const closeDropdown = () => {
    isDropdownOpen.value = false;
};
</script>

<template>
    <SidebarMenu>
        <SidebarMenuItem>
            <DropdownMenu v-model:open="isDropdownOpen">
                <DropdownMenuTrigger as-child>
                    <SidebarMenuButton
                        size="lg"
                        class="data-[state=open]:bg-white/10 data-[state=open]:text-white flex items-center gap-2 px-1 py-1.5 text-left text-sm transition-all duration-200 border-0 shadow-none p-3 rounded-xl"
                    >
                        <UserInfo :user="user" :show-email="true" />
                        <ChevronsUpDown class="ml-auto size-4 text-white/70" />
                    </SidebarMenuButton>
                </DropdownMenuTrigger>
                <DropdownMenuContent
                    class="w-64 min-w-56 rounded-xl shadow-xl border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900 backdrop-blur-sm"
                    :side="isMobile ? 'bottom' : state === 'collapsed' ? 'left' : 'bottom'"
                    align="end"
                    :side-offset="8"
                >
                    <!-- Modern Header with Close Button -->
                    <div class="flex items-center justify-between p-4 border-b border-gray-100 dark:border-gray-800">
                        <div class="flex items-center space-x-3">
                            <div class="w-10 h-10 rounded-xl flex items-center justify-center gap-2 px-1 py-1.5 text-left text-white font-bold text-sm shadow-sm">
                                <UserInfo :user="user" :show-email="true" />
                            </div>
                            <div>
                                <h3 class="font-semibold text-sm text-gray-900 dark:text-gray-100">{{ user.name || 'User' }}</h3>
                                <p class="text-xs text-gray-500 dark:text-gray-400">{{ user.email || 'user@example.com' }}</p>
                            </div>
                        </div>
                        <!-- Close Button -->
                        <button
                            @click="closeDropdown"
                            class="p-1.5 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg transition-colors group"
                        >
                            <X class="w-4 h-4 text-gray-400 group-hover:text-gray-600 dark:group-hover:text-gray-300" />
                        </button>
                    </div>

                    <!-- Menu Content -->
                    <div class="p-2">
                        <UserMenuContent :user="user" />
                    </div>
                </DropdownMenuContent>
            </DropdownMenu>
        </SidebarMenuItem>
    </SidebarMenu>
</template>
