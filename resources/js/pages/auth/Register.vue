<script setup lang="ts">
import RegisteredUserController from '@/actions/App/Http/Controllers/Auth/RegisteredUserController';
import InputError from '@/components/InputError.vue';
import TextLink from '@/components/TextLink.vue';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { login } from '@/routes';
import { Form, Head } from '@inertiajs/vue3';
import { LoaderCircle, Eye, EyeOff } from 'lucide-vue-next';
import { ref } from 'vue';

const showPassword = ref(false);
const showPasswordConfirmation = ref(false);

const togglePasswordVisibility = () => {
    showPassword.value = !showPassword.value;
};

const togglePasswordConfirmationVisibility = () => {
    showPasswordConfirmation.value = !showPasswordConfirmation.value;
};
</script>

<template>
    <div class="min-h-screen flex flex-col lg:flex-row">
        <Head title="Create Account - Messaging App" />

        <div class="flex-1 lg:flex hidden items-center justify-center p-4 lg:p-8 relative overflow-hidden bg-gradient-to-br from-teal-500 via-teal-600 to-emerald-600">

            <div class="max-w-md text-center text-white relative z-10">
                <div class="mb-8">
                    <h1 class="text-4xl lg:text-5xl font-bold mb-4 text-white">
                        Welcome Back!
                    </h1>
                </div>

                <p class="text-lg text-white/90 mb-8 leading-relaxed">
                    Access your account by signing in with your personal information.
                </p>

                <!-- Sign In Button -->
                <Button class="bg-transparent hover:bg-white/10 text-white border-2 border-white/50 hover:border-white/70 rounded-full px-12 py-3 font-semibold transition-all duration-300">
                    <TextLink :href="login()" class="text-white no-underline">
                        SIGN IN
                    </TextLink>
                </Button>
            </div>
        </div>

        <!-- Right side - Register form on white background -->
        <div class="flex-1 flex items-center justify-center p-8 bg-white">
            <div class="w-full max-w-md">
                <div class="text-center mb-8">
                    <h2 class="text-3xl font-bold text-teal-600 mb-2">Create Account</h2>
                    <div class="flex justify-center gap-4 mb-6">
                        <!-- Social login icons -->
                        <div class="w-10 h-10 border border-gray-300 rounded-full flex items-center justify-center cursor-pointer hover:bg-gray-50 transition-colors">
                            <span class="text-sm font-bold text-gray-600">f</span>
                        </div>
                        <div class="w-10 h-10 border border-gray-300 rounded-full flex items-center justify-center cursor-pointer hover:bg-gray-50 transition-colors">
                            <span class="text-sm font-bold text-gray-600">G+</span>
                        </div>
                        <div class="w-10 h-10 border border-gray-300 rounded-full flex items-center justify-center cursor-pointer hover:bg-gray-50 transition-colors">
                            <span class="text-sm font-bold text-gray-600">in</span>
                        </div>
                    </div>
                    <p class="text-gray-500 text-sm mb-6">or use your email for registration:</p>
                </div>

                <Form
                    v-bind="RegisteredUserController.store.form()"
                    :reset-on-success="['password', 'password_confirmation']"
                    v-slot="{ errors, processing }"
                    class="space-y-4"
                >
                    <!-- Name  -->
                    <div class="relative">
                        <Input
                            id="name"
                            type="text"
                            name="name"
                            required
                            autofocus
                            :tabindex="1"
                            autocomplete="name"
                            placeholder="Name"
                            class="h-12 pl-5 bg-gray-50 border-0 text-gray-700 placeholder-gray-400 focus:bg-white focus:ring-2 focus:ring-teal-500 rounded-lg transition-all"
                        />
                        <InputError :message="errors.name" class="text-red-500 text-sm mt-1" />
                    </div>

                    <!-- Email -->
                    <div class="relative">
                        <Input
                            id="email"
                            type="email"
                            name="email"
                            required
                            :tabindex="2"
                            autocomplete="email"
                            placeholder="Email"
                            class="h-12 pl-5 bg-gray-50 border-0 text-gray-700 placeholder-gray-400 focus:bg-white focus:ring-2 focus:ring-teal-500 rounded-lg transition-all"
                        />
                        <InputError :message="errors.email" class="text-red-500 text-sm mt-1" />
                    </div>

                    <!-- Password -->
                    <div class="relative">
                        <Input
                            id="password"
                            :type="showPassword ? 'text' : 'password'"
                            name="password"
                            required
                            :tabindex="3"
                            autocomplete="new-password"
                            placeholder="Password"
                            class="h-12 pl-5 pr-12 bg-gray-50 border-0 text-gray-700 placeholder-gray-700 focus:bg-white focus:ring-2 focus:ring-teal-500 rounded-lg transition-all"
                        />
                        <button
                            type="button"
                            @click="togglePasswordVisibility"
                            class="absolute right-4 top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600 transition-colors"
                            :tabindex="-1"
                        >
                            <Eye v-if="!showPassword" class="h-5 w-5" />
                            <EyeOff v-else class="h-5 w-5" />
                        </button>
                        <InputError :message="errors.password" class="text-red-500 text-sm mt-1" />
                    </div>

                    <!-- Password confirmation field -->
                    <div class="relative">
                        <div class="absolute left-4 top-1/2 transform -translate-y-1/2 h-5 w-5 text-gray-400">

                        </div>
                        <Input
                            id="password_confirmation"
                            :type="showPasswordConfirmation ? 'text' : 'password'"
                            name="password_confirmation"
                            required
                            :tabindex="4"
                            autocomplete="new-password"
                            placeholder="Confirm Password"
                            class="h-12 pl-5 pr-12 bg-gray-50 border-0 text-gray-700 placeholder-gray-400 focus:bg-white focus:ring-2 focus:ring-teal-500 rounded-lg transition-all"
                        />
                        <button
                            type="button"
                            @click="togglePasswordConfirmationVisibility"
                            class="absolute right-4 top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600 transition-colors"
                            :tabindex="-1"
                        >
                            <Eye v-if="!showPasswordConfirmation" class="h-5 w-5" />
                            <EyeOff v-else class="h-5 w-5" />
                        </button>
                        <InputError :message="errors.password_confirmation" class="text-red-500 text-sm mt-1" />
                    </div>

                    <!-- Submit button -->
                    <Button
                        type="submit"
                        class="w-full h-12 bg-gradient-to-r from-teal-500 to-teal-600 hover:from-teal-600 hover:to-teal-700 text-white font-semibold transition-all duration-300 rounded-full shadow-lg hover:shadow-xl transform hover:scale-[1.02] mt-6"
                        :tabindex="5"
                        :disabled="processing"
                    >
                        <LoaderCircle v-if="processing" class="h-5 w-5 animate-spin mr-2" />
                        <span v-else>SIGN UP</span>
                    </Button>
                </Form>
            </div>
        </div>
    </div>
</template>

<style scoped>

input {
    background: #F9FAFB;
    border: none;
}
input::placeholder {
    color: gray;
}

input:focus {
    background: white;
    box-shadow: 0 0 0 2px #14B8A6;
    outline: none;
}

* {
    transition-duration: 200ms;
    transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
</style>
